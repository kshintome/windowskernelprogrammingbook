# DelProtect - ファイル削除保護ミニフィルタドライバ

## 概要

DelProtectは、cmd.exeからのファイル削除操作を防止するためのWindowsカーネルモードミニフィルタドライバです。このドライバは、ファイルシステムとの間に入り、特定のプロセスからのファイル削除試行を監視・ブロックします。

## 動作原理

1. **削除操作の検出**
   - `IRP_MJ_CREATE`操作で`FILE_DELETE_ON_CLOSE`フラグを監視
   - `IRP_MJ_SET_INFORMATION`操作で`FileDispositionInformation`と`FileDispositionInformationEx`を監視

2. **プロセスの識別**
   - `ZwQueryInformationProcess`を使用してプロセス名を取得
   - `cmd.exe`（System32またはSysWOW64フォルダ内）からの削除操作を特定

3. **削除操作のブロック**
   - cmd.exeからの削除要求を検出した場合、`STATUS_ACCESS_DENIED`を返却
   - 他のプロセスからの削除操作は通常通り処理される

## 主要コンポーネント

- **DelProtectPreCreate**: ファイル作成時の削除フラグをチェック
- **DelProtectPreSetInformation**: ファイル情報設定時の削除操作をチェック
- **IsDeleteAllowed**: プロセス名をチェックし、cmd.exeからの削除を拒否

## 技術的詳細

- Windowsフィルタマネージャ（FltMgr）を使用したファイルシステムミニフィルタ実装
- カーネルモードで動作し、ユーザーモードアプリケーションから透過的に機能
- 操作のフィルタリングにプレオペレーションコールバックを利用