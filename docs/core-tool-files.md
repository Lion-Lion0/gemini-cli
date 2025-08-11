# packages/core と packages/core/src/tools のファイル構成

このドキュメントでは、Gemini CLI の **core** パッケージ (`packages/core`) と
その配下にある **tools** ディレクトリ (`packages/core/src/tools`) の主要ファイルの
役割と相互関係を解説します。

## ディレクトリ構成

```mermaid
flowchart TD
  A[packages/core]
  A --> B(config)
  A --> C(core)
  A --> D(services)
  A --> E(tools)
  A --> F(utils)
  A --> G(telemetry)
```

- **config**: 設定読み込みやモデル名定義を行う。
- **core**: Gemini API とのやり取りやチャット管理ロジックを含む。
- **services**: ファイル探索や Git 連携など補助サービス群。
- **tools**: ツールの定義と実装。
- **utils**: 各種ユーティリティ関数。
- **telemetry**: ログ出力やセッション管理。

## コア機能の関係

```mermaid
flowchart TD
  subgraph Core
    client[client.ts]
    contentGenerator[contentGenerator.ts]
    geminiChat[geminiChat.ts]
    geminiRequest[geminiRequest.ts]
    toolScheduler[coreToolScheduler.ts]
  end
  cli[CLI からの入力]
  toolsDir[tools/]

  cli --> geminiRequest
  geminiRequest --> contentGenerator
  contentGenerator --> geminiChat
  geminiChat --> toolScheduler
  toolScheduler --> toolsDir
```

- **client.ts**: API 呼び出し用の基本クラス。
- **geminiRequest.ts**: CLI からのリクエストを処理し、`contentGenerator`へ渡す。
- **contentGenerator.ts**: プロンプト生成や API 呼び出し設定を担当。
- **geminiChat.ts**: チャット履歴管理と応答の整形。
- **coreToolScheduler.ts**: モデルから要求されたツール呼び出しを調整する。

## tools ディレクトリ

```mermaid
flowchart TD
  subgraph Tools
    toolsTs[tools.ts]
    registry[tool-registry.ts]
    ls[ls.ts]
    readFile[read-file.ts]
    writeFile[write-file.ts]
    grep[grep.ts]
    glob[glob.ts]
    edit[edit.ts]
    shell[shell.ts]
    webFetch[web-fetch.ts]
    webSearch[web-search.ts]
    memory[memoryTool.ts]
    readMany[read-many-files.ts]
    mcpClient[mcp-client.ts]
    mcpTool[mcp-tool.ts]
  end
  toolsTs --> registry
  registry --> ls
  registry --> readFile
  registry --> writeFile
  registry --> grep
  registry --> glob
  registry --> edit
  registry --> shell
  registry --> webFetch
  registry --> webSearch
  registry --> memory
  registry --> readMany
  registry --> mcpClient
  registry --> mcpTool
```

- **tools.ts**: `BaseTool` や `Tool` インターフェースなど、ツール実装の基底となる定義を提供。
- **tool-registry.ts**: ツールの登録・発見・取得を管理するクラス。
- **ls.ts** など各種ファイルは具体的なツールの実装。
- **mcp-client.ts / mcp-tool.ts**: MCP サーバーから取得したツールを扱うためのロジック。

## ツール呼び出しの流れ

```mermaid
sequenceDiagram
  participant CLI
  participant Core
  participant Registry as ToolRegistry
  participant Tool
  CLI->>Core: ユーザー入力
  Core->>Registry: 使用可能なツール一覧を取得
  Core->>CLI: 必要に応じ確認
  CLI->>Core: 実行許可
  Core->>Tool: execute(params)
  Tool-->>Core: ToolResult
  Core-->>CLI: 表示用の結果
```

各ツールは `BaseTool` を継承し、`execute()` メソッドで処理を実装します。
`ToolRegistry` は利用可能なツールを管理し、モデルからのリクエストに応じて
対象ツールを呼び出します。

