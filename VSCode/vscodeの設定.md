# workspace

# 自動フォーマット

```json
settings.json
{
  // ...
  "[typescript]": {
    "editor.formatOnSave": true,
    "editor.formatOnPaste": true
  },
  "[markdown]": {
    "editor.formatOnSave": true,
    "editor.wordWrap": "on",
    "editor.renderWhitespace": "all",
    "editor.acceptSuggestionOnEnter": "off"
  },
  "[php]": {
    "editor.defaultFormatter": "junstyle.php-cs-fixer"
  },
}
```

# タスクの自動実行

# チーム開発

## 拡張機能の共有

.vscode/extensions.json

## Remote Containers

- `.devcontainer`フォルダ
- `devcontainer.json`

### git の有効化

- `devcontainer.json`
  - `postCreateCommand`に記述したコマンドは、コンテナ作成後に自動で実行されます。
  - コマンドはコンテナに入っているパッケージマネージャーのコマンドに置き換えて下さい。

```json
// Uncomment the next line to run commands after the container is created - for example installing git.
"postCreateCommand": "apt update && apt install -y git",
```
