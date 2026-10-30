🌏 **繁體中文** ｜ [English 原文](https://github.com/gitleaks/gitleaks)

> **非官方翻譯**：本 repo 是 [gitleaks/gitleaks](https://github.com/gitleaks/gitleaks) README 的中文翻譯，與原作者無關，亦非官方出品。
> 原專案授權：MIT（見 [LICENSE](LICENSE)）。翻譯依據原專案 commit `5f6ec56`（2026-10-30），內容可能落後原文，請以原專案為準。
> 📚 想從零上手？[中文實戰教程](https://vocus.cc/salon/Henry)

---

# Gitleaks

```
┌─○───┐
│ │╲  │
│ │ ○ │
│ ○ ░ │
└─░───┘
```

> [!WARNING]
> Gitleaks 功能已經完備。我不會再將新功能合併進 Gitleaks，未來的版本將只提供安全性修補。我會將重心轉移到 [Betterleaks](https://github.com/betterleaks/betterleaks)

[license]: https://github.com/gitleaks/gitleaks/blob/master/LICENSE
[badge-license]: https://img.shields.io/github/license/gitleaks/gitleaks.svg
[go-docs-badge]: https://pkg.go.dev/badge/github.com/gitleaks/gitleaks/v8?status
[go-docs]: https://pkg.go.dev/github.com/zricethezav/gitleaks/v8
[badge-build]: https://github.com/gitleaks/gitleaks/actions/workflows/test.yml/badge.svg
[build]: https://github.com/gitleaks/gitleaks/actions/workflows/test.yml
[go-report-card-badge]: https://goreportcard.com/badge/github.com/gitleaks/gitleaks/v8
[go-report-card]: https://goreportcard.com/report/github.com/gitleaks/gitleaks/v8
[dockerhub]: https://hub.docker.com/r/zricethezav/gitleaks
[dockerhub-badge]: https://img.shields.io/docker/pulls/zricethezav/gitleaks.svg
[gitleaks-action]: https://github.com/gitleaks/gitleaks-action
[gitleaks-badge]: https://img.shields.io/badge/protected%20by-gitleaks-blue


[![GitHub Action Test][badge-build]][build]
[![Docker Hub][dockerhub-badge]][dockerhub]
[![Gitleaks Action][gitleaks-badge]][gitleaks-action]
[![GoDoc][go-docs-badge]][go-docs]
[![GoReportCard][go-report-card-badge]][go-report-card]
[![License][badge-license]][license]

Gitleaks 是一個用來**偵測**機密資訊的工具，例如 git 儲存庫、檔案中的密碼、API 金鑰與權杖（token），以及任何你想透過 `stdin` 丟給它的內容。如果你想進一步了解偵測引擎的運作方式，請參考這篇部落格文章：[Regex is (almost) all you need](https://lookingatcomputer.substack.com/p/regex-is-almost-all-you-need)。

```
➜  ~/code(master) gitleaks git -v

    ○
    │╲
    │ ○
    ○ ░
    ░    gitleaks


Finding:     "export BUNDLE_ENTERPRISE__CONTRIBSYS__COM=cafebabe:deadbeef",
Secret:      cafebabe:deadbeef
RuleID:      sidekiq-secret
Entropy:     2.609850
File:        cmd/generate/config/rules/sidekiq.go
Line:        23
Commit:      cd5226711335c68be1e720b318b7bc3135a30eb2
Author:      John
Email:       john@users.noreply.github.com
Date:        2022-08-03T12:31:40Z
Fingerprint: cd5226711335c68be1e720b318b7bc3135a30eb2:cmd/generate/config/rules/sidekiq.go:sidekiq-secret:23
```


## Getting Started

Gitleaks 可以透過 Homebrew、Docker 或 Go 安裝。Gitleaks 也在[發行頁面](https://github.com/gitleaks/gitleaks/releases)提供適用於許多主流平台與作業系統的二進位檔。此外，Gitleaks 可以直接在你的儲存庫中作為 pre-commit hook 使用，或是透過 [Gitleaks-Action](https://github.com/gitleaks/gitleaks-action) 作為 GitHub Action 使用。

### Installing

```bash
# MacOS
brew install gitleaks

# Docker (DockerHub)
docker pull zricethezav/gitleaks:latest
docker run -v ${path_to_host_folder_to_scan}:/path zricethezav/gitleaks:latest [COMMAND] [OPTIONS] [SOURCE_PATH]

# Docker (ghcr.io)
docker pull ghcr.io/gitleaks/gitleaks:latest
docker run -v ${path_to_host_folder_to_scan}:/path ghcr.io/gitleaks/gitleaks:latest [COMMAND] [OPTIONS] [SOURCE_PATH]

# From Source (make sure `go` is installed)
git clone https://github.com/gitleaks/gitleaks.git
cd gitleaks
make build
```

### Pre-Commit

1. 從 https://pre-commit.com/#install 安裝 pre-commit
2. 在儲存庫的根目錄建立 `.pre-commit-config.yaml` 檔案，內容如下：

   ```
   repos:
     - repo: https://github.com/gitleaks/gitleaks
       rev: v8.24.2
       hooks:
         - id: gitleaks
   ```

   若要[以原生方式執行 gitleaks](https://github.com/gitleaks/gitleaks/releases)，請使用上述設定；若要使用[官方 Docker 映像檔](#docker)執行 gitleaks，則請使用 [`gitleaks-docker` pre-commit ID](https://github.com/gitleaks/gitleaks/blob/master/.pre-commit-hooks.yaml)

3. 執行 `pre-commit autoupdate`，將設定自動更新為各儲存庫的最新版本
4. 執行 `pre-commit install` 進行安裝
5. 這樣就全部設定完成了！

```
➜ git commit -m "this commit contains a secret"
Detect hardcoded secrets.................................................Failed
```

注意：若要停用 gitleaks 的 pre-commit hook，可以在 commit 指令前加上 `SKIP=gitleaks`，
這樣就會跳過執行 gitleaks

```
➜ SKIP=gitleaks git commit -m "skip gitleaks check"
Detect hardcoded secrets................................................Skipped
```

## Usage

```
Gitleaks scans code, past or present, for secrets

Usage:
  gitleaks [command]

Available Commands:
  completion  Generate the autocompletion script for the specified shell
  dir         scan directories or files for secrets
  git         scan git repositories for secrets
  help        Help about any command
  stdin       detect secrets from stdin
  version     display gitleaks version

Flags:
  -b, --baseline-path string          path to baseline with issues that can be ignored
  -c, --config string                 config file path
                                      order of precedence:
                                      1. --config/-c
                                      2. env var GITLEAKS_CONFIG
                                      3. env var GITLEAKS_CONFIG_TOML with the file content
                                      4. (target path)/.gitleaks.toml
                                      If none of the four options are used, then gitleaks will use the default config
      --diagnostics string            enable diagnostics (http OR comma-separated list: cpu,mem,trace). cpu=CPU prof, mem=memory prof, trace=exec tracing, http=serve via net/http/pprof
      --diagnostics-dir string        directory to store diagnostics output files when not using http mode (defaults to current directory)
      --enable-rule strings           only enable specific rules by id
      --exit-code int                 exit code when leaks have been encountered (default 1)
  -i, --gitleaks-ignore-path string   path to .gitleaksignore file or folder containing one (default ".")
  -h, --help                          help for gitleaks
      --ignore-gitleaks-allow         ignore gitleaks:allow comments
  -l, --log-level string              log level (trace, debug, info, warn, error, fatal) (default "info")
      --max-archive-depth int         allow scanning into nested archives up to this depth (default "0", no archive traversal is done)
      --max-decode-depth int          allow recursive decoding up to this depth (default "0", no decoding is done)
      --max-target-megabytes int      files larger than this will be skipped
      --no-banner                     suppress banner
      --no-color                      turn off color for verbose output
      --redact uint[=100]             redact secrets from logs and stdout. To redact only parts of the secret just apply a percent value from 0..100. For example --redact=20 (default 100%)
  -f, --report-format string          output format (json, csv, junit, sarif, template)
  -r, --report-path string            report file
      --report-template string        template file used to generate the report (implies --report-format=template)
      --timeout int                   set a timeout for gitleaks commands in seconds (default "0", no timeout is set)
  -v, --verbose                       show verbose output from scan
      --version                       version for gitleaks

Use "gitleaks [command] --help" for more information about a command.
```

### Commands

⚠️ v8.19.0 引入了一項變更，將 `detect` 與 `protect` 標示為已棄用。這些指令仍然可用，但在 `--help` 選單中已被隱藏。請參考這份 [gist](https://gist.github.com/zricethezav/b325bb93ebf41b9c0b0507acf12810d2)，以便輕鬆對照新舊指令。
如果你發現 v8.19.0 破壞了既有的指令（`detect`／`protect`），請開一個 issue。

共有三種掃描模式：`git`、`dir` 與 `stdin`。

#### Git

`git` 指令可讓你掃描本機的 git 儲存庫。在底層，gitleaks 使用 `git log -p` 指令來掃描修補內容（patch）。
你可以透過 `log-opts` 選項來設定 `git log -p` 的行為。
例如，若想對某個範圍內的 commit 執行 gitleaks，可以使用以下指令：`gitleaks git -v --log-opts="--all commitA..commitB" path_to_repo`。更多資訊請參閱 [git log](https://git-scm.com/docs/git-log) 文件。
如果沒有以位置引數指定目標，gitleaks 會嘗試將目前的工作目錄當作 git 儲存庫來掃描。

#### Dir

`dir` 指令（別名包含 `files`、`directory`）可讓你掃描目錄與檔案。範例：`gitleaks dir -v path_to_directory_or_file`。
如果沒有以位置引數指定目標，gitleaks 會掃描目前的工作目錄。

#### Stdin

你也可以使用 `stdin` 指令將資料以串流方式傳給 gitleaks。範例：`cat some_file | gitleaks -v stdin`

### Creating a baseline

在掃描大型儲存庫或歷史悠久的儲存庫時，使用基準線（baseline）會很方便。使用基準線時，
gitleaks 會忽略基準線中已存在的所有舊有發現。基準線可以是任何一份 gitleaks 報告。要建立 gitleaks 報告，請在執行 gitleaks 時加上 `--report-path` 參數。

```
gitleaks git --report-path gitleaks-report.json # This will save the report in a file called gitleaks-report.json
```

建立基準線之後，就可以在再次執行 detect 指令時套用它：

```
gitleaks git --baseline-path gitleaks-report.json --report-path findings.json
```

在使用 --baseline-path 參數執行 detect 指令之後，報告輸出（findings.json）將只包含新的問題。

## Pre-Commit hook

你可以將範例 `pre-commit.py` 指令碼複製到你的 `.git/hooks/` 目錄中，
讓 Gitleaks 作為 pre-commit hook 執行。

## Load Configuration

優先順序如下：

1. `--config/-c` 選項：
      ```bash
      gitleaks git --config /home/dev/customgitleaks.toml .
      ```
2. 以檔案路徑設定環境變數 `GITLEAKS_CONFIG`：
      ```bash
      export GITLEAKS_CONFIG="/home/dev/customgitleaks.toml"
      gitleaks git .
      ```
3. 以檔案內容設定環境變數 `GITLEAKS_CONFIG_TOML`：
      ```bash
      export GITLEAKS_CONFIG_TOML=`cat customgitleaks.toml`
      gitleaks git .
      ```
4. 目標路徑中的 `.gitleaks.toml` 檔案：
      ```bash
      gitleaks git .
      ```

如果四種選項都沒有使用，gitleaks 就會使用預設設定。

## 設定

Gitleaks 提供一種設定格式，您可以依照它來撰寫自己的機密偵測規則：

```toml
# Title for the gitleaks configuration file.
title = "Custom Gitleaks configuration"

# You have basically two options for your custom configuration:
#
# 1. define your own configuration, default rules do not apply
#
#    use e.g., the default configuration as starting point:
#    https://github.com/gitleaks/gitleaks/blob/master/config/gitleaks.toml
#
# 2. extend a configuration, the rules are overwritten or extended
#
#    When you extend a configuration the extended rules take precedence over the
#    default rules. I.e., if there are duplicate rules in both the extended
#    configuration and the default configuration the extended rules or
#    attributes of them will override the default rules.
#    Another thing to know with extending configurations is you can chain
#    together multiple configuration files to a depth of 2. Allowlist arrays are
#    appended and can contain duplicates.

# useDefault and path can NOT be used at the same time. Choose one.
[extend]
# useDefault will extend the default gitleaks config built in to the binary
# the latest version is located at:
# https://github.com/gitleaks/gitleaks/blob/master/config/gitleaks.toml
useDefault = true
# or you can provide a path to a configuration to extend from.
# The path is relative to where gitleaks was invoked,
# not the location of the base config.
# path = "common_config.toml"
# If there are any rules you don't want to inherit, they can be specified here.
disabledRules = [ "generic-api-key"]

# An array of tables that contain information that define instructions
# on how to detect secrets
[[rules]]
# Unique identifier for this rule
id = "awesome-rule-1"

# Short human-readable description of the rule.
description = "awesome rule 1"

# Golang regular expression used to detect secrets. Note Golang's regex engine
# does not support lookaheads.
regex = '''one-go-style-regex-for-this-rule'''

# Int used to extract secret from regex match and used as the group that will have
# its entropy checked if `entropy` is set.
secretGroup = 3

# Float representing the minimum shannon entropy a regex group must have to be considered a secret.
entropy = 3.5

# Golang regular expression used to match paths. This can be used as a standalone rule or it can be used
# in conjunction with a valid `regex` entry.
path = '''a-file-path-regex'''

# Keywords are used for pre-regex check filtering. Rules that contain
# keywords will perform a quick string compare check to make sure the
# keyword(s) are in the content being scanned. Ideally these values should
# either be part of the identiifer or unique strings specific to the rule's regex
# (introduced in v8.6.0)
keywords = [
  "auth",
  "password",
  "token",
]

# Array of strings used for metadata and reporting purposes.
tags = ["tag","another tag"]

    # ⚠️ In v8.21.0 `[rules.allowlist]` was replaced with `[[rules.allowlists]]`.
    # This change was backwards-compatible: instances of `[rules.allowlist]` still  work.
    #
    # You can define multiple allowlists for a rule to reduce false positives.
    # A finding will be ignored if _ANY_ `[[rules.allowlists]]` matches.
    [[rules.allowlists]]
    description = "ignore commit A"
    # When multiple criteria are defined the default condition is "OR".
    # e.g., this can match on |commits| OR |paths| OR |stopwords|.
    condition = "OR"
    commits = [ "commit-A", "commit-B"]
    paths = [
      '''go\.mod''',
      '''go\.sum'''
    ]
    # note: stopwords targets the extracted secret, not the entire regex match
    # like 'regexes' does. (stopwords introduced in 8.8.0)
    stopwords = [
      '''client''',
      '''endpoint''',
    ]

    [[rules.allowlists]]
    # The "AND" condition can be used to make sure all criteria match.
    # e.g., this matches if |regexes| AND |paths| are satisfied.
    condition = "AND"
    # note: |regexes| defaults to check the _Secret_ in the finding.
    # Acceptable values for |regexTarget| are "secret" (default), "match", and "line".
    regexTarget = "match"
    regexes = [ '''(?i)parseur[il]''' ]
    paths = [ '''package-lock\.json''' ]

# You can extend a particular rule from the default config. e.g., gitlab-pat
# if you have defined a custom token prefix on your GitLab instance
[[rules]]
id = "gitlab-pat"
# all the other attributes from the default rule are inherited

    [[rules.allowlists]]
    regexTarget = "line"
    regexes = [ '''MY-glpat-''' ]


# ⚠️ In v8.25.0 `[allowlist]` was replaced with `[[allowlists]]`.
#
# Global allowlists have a higher order of precedence than rule-specific allowlists.
# If a commit listed in the `commits` field below is encountered then that commit will be skipped and no
# secrets will be detected for said commit. The same logic applies for regexes and paths.
[[allowlists]]
description = "global allow list"
commits = [ "commit-A", "commit-B", "commit-C"]
paths = [
  '''gitleaks\.toml''',
  '''(.*?)(jpg|gif|doc)'''
]
# note: (global) regexTarget defaults to check the _Secret_ in the finding.
# Acceptable values for regexTarget are "match" and "line"
regexTarget = "match"
regexes = [
  '''219-09-9999''',
  '''078-05-1120''',
  '''(9[0-9]{2}|666)-\d{2}-\d{4}''',
]
# note: stopwords targets the extracted secret, not the entire regex match
# like 'regexes' does. (stopwords introduced in 8.8.0)
stopwords = [
  '''client''',
  '''endpoint''',
]

# ⚠️ In v8.25.0, `[[allowlists]]` have a new field called |targetRules|.
#
# Common allowlists can be defined once and assigned to multiple rules using |targetRules|.
# This will only run on the specified rules, not globally.
[[allowlists]]
targetRules = ["awesome-rule-1", "awesome-rule-2"]
description = "Our test assets trigger false-positives in a couple rules."
paths = ['''tests/expected/._\.json$''']
```

請參考預設的 [gitleaks config](https://github.com/gitleaks/gitleaks/blob/master/config/gitleaks.toml) 以取得範例；若您想為預設設定做出貢獻，請遵循[貢獻指南](https://github.com/gitleaks/gitleaks/blob/master/CONTRIBUTING.md)。此外，您也可以參考[這篇 gitleaks 部落格文章](https://blog.gitleaks.io/stop-leaking-secrets-configuration-2-3-aeed293b1fbf)，其中介紹了進階的設定方式。

### 其他設定

#### 複合規則（Composite Rules，多部分或 `required` 規則）
Gitleaks 在 v8.28.0 引入了複合規則，由一個「主要」規則與一個或多個輔助規則（即 `required` 規則）組成。若要建立複合規則，請在主要規則中加入 `[[rules.required]]` 表格，指定 `id`，並可選擇性地加上 `withinLines` 和／或 `withinColumns` 鄰近限制。片段（fragment）是 Gitleaks 一次處理的一塊內容（通常是一個檔案、檔案的一部分，或 git diff），而鄰近比對（proximity matching）會指示主要規則，只有在輔助的 `required` 規則也於片段中的指定範圍內找到符合項目時，才回報發現結果。

**鄰近比對：** 使用 `withinLines` 與 `withinColumns` 欄位，可指示主要規則僅在輔助的 `required` 規則也於指定的鄰近範圍內找到符合項目時，才回報發現結果。你可以設定：

- **`withinLines: N`** - required 的發現結果必須在 N 行之內（垂直方向）
- **`withinColumns: N`** - required 的發現結果必須在 N 個字元之內（水平方向）
- **兩者皆設定** - 建立矩形搜尋區域（兩項限制都必須滿足）
- **兩者皆不設定** - 片段層級比對（required 的發現結果可位於同一個片段中的任何位置）

以下圖示說明各種鄰近行為：

```
p = primary captured secret
a = auxiliary (required) captured secret
fragment = section of data gitleaks is looking at


    *Fragment-level proximity*
    Any required finding in the fragment
          ┌────────┐
   ┌──────┤fragment├─────┐
   │      └──────┬─┤     │ ┌───────┐
   │             │a│◀────┼─│✓ MATCH│
   │          ┌─┐└─┘     │ └───────┘
   │┌─┐       │p│        │
   ││a│    ┌─┐└─┘        │ ┌───────┐
   │└─┘    │a│◀──────────┼─│✓ MATCH│
   └─▲─────┴─┴───────────┘ └───────┘
     │    ┌───────┐
     └────│✓ MATCH│
          └───────┘


   *Column bounded proximity*
   `withinColumns = 3`
          ┌────────┐
   ┌────┬─┤fragment├─┬───┐
   │      └──────┬─┤     │ ┌───────────┐
   │    │        │a│◀┼───┼─│+1C ✓ MATCH│
   │          ┌─┐└─┘     │ └───────────┘
   │┌─┐ │     │p│    │   │
┌──▶│a│  ┌─┐  └─┘        │ ┌───────────┐
│  │└─┘ ││a│◀────────┼───┼─│-2C ✓ MATCH│
│  │       ┘             │ └───────────┘
│  └── -3C ───0C─── +3C ─┘
│  ┌─────────┐
│  │ -4C ✗ NO│
└──│  MATCH  │
   └─────────┘


   *Line bounded proximity*
   `withinLines = 4`
         ┌────────┐
   ┌─────┤fragment├─────┐
  +4L─ ─ ┴────────┘─ ─ ─│
   │                    │
   │              ┌─┐   │ ┌────────────┐
   │         ┌─┐  │a│◀──┼─│+1L ✓ MATCH │
   0L  ┌─┐   │p│  └─┘   │ ├────────────┤
   │   │a│◀──┴─┴────────┼─│-1L ✓ MATCH │
   │   └─┘              │ └────────────┘
   │                    │ ┌─────────┐
  -4L─ ─ ─ ─ ─ ─ ─ ─┌─┐─│ │-5L ✗ NO │
   │                │a│◀┼─│  MATCH  │
   └────────────────┴─┴─┘ └─────────┘


   *Line and column bounded proximity*
   `withinLines = 4`
   `withinColumns = 3`
         ┌────────┐
   ┌─────┤fragment├─────┐
  +4L   ┌└────────┴ ┐   │
   │            ┌─┐     │ ┌───────────────┐
   │    │       │a│◀┼───┼─│+2L/+1C ✓ MATCH│
   │         ┌─┐└─┘     │ └───────────────┘
   0L   │    │p│    │   │
   │         └─┘        │
   │    │           │   │ ┌────────────┐
  -4L    ─ ─ ─ ─ ─ ─┌─┐ │ │-5L/+3C ✗ NO│
   │                │a│◀┼─│   MATCH    │
   └───-3C────0L───+3C┴─┘ └────────────┘
```

<details><summary>關於複合規則的一些最後想法。</summary>這是實驗性功能！它可能會變動，所以別拿這個功能去打造新的 B2B SaaS 功能然後拿出去賣。依掃描類型（git 與 dir）而定的情境很有意思，我正在持續觀察。複合規則對 git 掃描可能不是那麼有用，因為 gitleaks 只會查看 git 歷史中新增的內容。對 `required` 規則掃描 git 歷史中非新增的內容，或許會有用。啊，對了，這是 README，我就不囉嗦了。</details>

#### gitleaks:allow

如果你明知自己提交的是測試用的密鑰，而 gitleaks 會偵測到它，可以在該行加上 `gitleaks:allow` 註解，指示 gitleaks
忽略該密鑰。例如：

```
class CustomClass:
    discord_client_secret = '8dyfuiRyq=vVc3RRr_edRk-fK__JItpZ'  #gitleaks:allow

```

#### .gitleaksignore

你可以在儲存庫根目錄建立 `.gitleaksignore` 檔案，以忽略特定的發現結果。Gitleaks 在 v8.10.0 版本中為報告新增了 `Fingerprint` 值。每個洩漏（發現結果）都有一個可唯一識別該密鑰的 Fingerprint。將此 fingerprint 加入 `.gitleaksignore` 檔案，即可忽略該特定密鑰。範例請參考 Gitleaks 的 [.gitleaksignore](https://github.com/gitleaks/gitleaks/blob/master/.gitleaksignore)。注意：此功能為實驗性質，日後可能會變動。

#### 解碼

有時密鑰經過編碼，僅靠正規表達式很難找到。現在你可以讓 gitleaks 自動尋找並解碼
已編碼的文字。旗標 `--max-decode-depth` 可啟用此功能（預設值
「0」表示預設停用此功能）。

由於解碼後的文字也可能包含已編碼的文字，因此支援遞迴解碼。旗標 `--max-decode-depth` 用於設定遞迴上限。當沒有新的已編碼文字片段需要解碼時，遞迴就會停止，
因此把最大深度設得很高，並不代表會執行那麼多輪。它只會執行解碼文字所需的
次數。整體而言，解碼只會小幅增加
掃描時間。

已編碼文字的發現結果與一般發現結果在以下方面有所不同：

- 位置會指向已編碼文字的範圍
  - 如果規則比對到的內容超出已編碼文字之外，範圍會調整為
    也包含該部分
- 比對內容（match）與密鑰（secret）包含解碼後的值
- 會加上兩個標籤：`decoded:<encoding>` 與 `decode-depth:<depth>`

目前支援的編碼：

- **percent** - 任何可列印 ASCII 的百分比編碼（percent-encoded）值
- **hex** - 任何長度 >= 32 個字元的可列印 ASCII 十六進位編碼值
- **base64** - 任何長度 >= 16 個字元的可列印 ASCII base64 編碼值

#### 封存檔掃描

有時密鑰會被封裝在 zip 檔或 tarball 等封存檔中，
導致難以發現。現在你可以讓 gitleaks 自動
解壓縮並掃描封存檔的內容。旗標 `--max-archive-depth`
可同時為 `dir` 與 `git` 掃描類型啟用此功能。預設值
「0」表示預設停用此功能。

由於封存檔中也可能包含其他封存檔，因此支援遞迴掃描。
旗標 `--max-archive-depth` 用於設定遞迴上限。當沒有新的封存檔需要解壓縮時，遞迴就會停止，
因此把最大深度設得很高，只是代表有可能深入到那個層數。實際上只會深入到所需的深度。

位於封存檔內的密鑰，其發現結果會包含封存檔內該檔案的路徑。內部路徑以 `!` 分隔。

發現結果範例（為求簡潔已縮短）：

```
Finding:     DB_PASSWORD=8ae31cacf141669ddfb5da
...
File:        testdata/archives/nested.tar.gz!archives/files.tar!files/.env.prod
Line:        4
Commit:      6e6ee6596d337bb656496425fb98644eb62b4a82
...
Fingerprint: 6e6ee6596d337bb656496425fb98644eb62b4a82:testdata/archives/nested.tar.gz!archives/files.tar!files/.env.prod:generic-api-key:4
Link:        https://github.com/leaktk/gitleaks/blob/6e6ee6596d337bb656496425fb98644eb62b4a82/testdata/archives/nested.tar.gz
```

這表示在 `files/.env.prod.` 的第 4 行偵測到密鑰，該檔案位於
`archives/files.tar` 中，而 `archives/files.tar` 位於 `testdata/archives/nested.tar.gz` 中。

目前支援的格式：

mholt 的 [archives package](https://github.com/mholt/archives) 所支援的
[壓縮](https://github.com/mholt/archives?tab=readme-ov-file#supported-compression-formats)
與[封存](https://github.com/mholt/archives?tab=readme-ov-file#supported-archive-formats)
格式皆受支援。

#### 報告

Gitleaks 內建支援多種報告格式：[`json`](https://github.com/gitleaks/gitleaks/blob/master/testdata/expected/report/json_simple.json)、[`csv`](https://github.com/gitleaks/gitleaks/blob/master/testdata/expected/report/csv_simple.csv?plain=1)、[`junit`](https://github.com/gitleaks/gitleaks/blob/master/testdata/expected/report/junit_simple.xml) 與 [`sarif`](https://github.com/gitleaks/gitleaks/blob/master/testdata/expected/report/sarif_simple.sarif)。

如果這些格式都不符合你的需求，可以使用 [Go `text/template` .tmpl 檔案](https://www.digitalocean.com/community/tutorials/how-to-use-templates-in-go#step-4-writing-a-template)與 `--report-template` 旗標建立自己的報告格式。範本可以使用 [`Masterminds/sprig` 範本函式庫的擴充功能](https://masterminds.github.io/sprig/)。

例如，以下範本提供自訂的 JSON 輸出：
```gotemplate
# jsonextra.tmpl
[{{ $lastFinding := (sub (len . ) 1) }}
{{- range $i, $finding := . }}{{with $finding}}
    {
        "Description": {{ quote .Description }},
        "StartLine": {{ .StartLine }},
        "EndLine": {{ .EndLine }},
        "StartColumn": {{ .StartColumn }},
        "EndColumn": {{ .EndColumn }},
        "Line": {{ quote .Line }},
        "Match": {{ quote .Match }},
        "Secret": {{ quote .Secret }},
        "File": "{{ .File }}",
        "SymlinkFile": {{ quote .SymlinkFile }},
        "Commit": {{ quote .Commit }},
        "Entropy": {{ .Entropy }},
        "Author": {{ quote .Author }},
        "Email": {{ quote .Email }},
        "Date": {{ quote .Date }},
        "Message": {{ quote .Message }},
        "Tags": [{{ $lastTag := (sub (len .Tags ) 1) }}{{ range $j, $tag := .Tags }}{{ quote . }}{{ if ne $j $lastTag }},{{ end }}{{ end }}],
        "RuleID": {{ quote .RuleID }},
        "Fingerprint": {{ quote .Fingerprint }}
    }{{ if ne $i $lastFinding }},{{ end }}
{{- end}}{{ end }}
]
```

用法：
```sh
$ gitleaks dir ~/leaky-repo/ --report-path "report.json" --report-format template --report-template testdata/report/jsonextra.tmpl
```

## 贊助

<p align="left">
	<h3><a href="https://coderabbit.ai/?utm_source=oss&utm_medium=sponsorship&utm_campaign=gitleaks">coderabbit.ai</h3>
	  <a href="https://coderabbit.ai/?utm_source=oss&utm_medium=sponsorship&utm_campaign=gitleaks">
		  <img alt="CodeRabbit.ai Sponsorship" src="https://github.com/gitleaks/gitleaks/assets/15034943/76c30a85-887b-47ca-9956-17a8e55c6c41" width=200>
	  </a>
</p>


## 結束代碼

遇到洩漏時，你隨時可以使用 --exit-code 旗標設定結束代碼。預設的結束代碼如下：

```
0 - no leaks present
1 - leaks or error encountered
126 - unknown flag
```

### 加入 Discord！ [![Discord](https://img.shields.io/discord/1102689410522284044.svg?label=&logo=discord&logoColor=ffffff&color=7389D8&labelColor=6A7EC2)](https://discord.gg/8Hzbrnkr7E)
