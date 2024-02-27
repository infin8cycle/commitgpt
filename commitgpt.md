The purpose of this document is to provide you with a tool for automatically generating commit messages.

As Git changes are executed on your local computer, please run the following commands within your local environment.


## Preparation
Install shell_gpt

```pip install shell-gpt```

Please update the configuration by adding your OpenAI API key to the ```~/.config/shell_gpt/.sgptrc``` file.

```
[...]
DEFAULT_MODEL=gpt-4
OPENAI_API_KEY=your-api-key
[...]
```

## Role
Introduce a new role called ```commitgpt``` which will serve as the ChatGPT prompt.
```
$ sgpt --create-role commitgpt
Enter role description: Compose a concise Git commit header with a maximum of 50 characters, don't add double quotes, summarizing the changes made. If necessary and relevant, provide additional details in bullet points below the header. Ensure that the additional details are wrapped at 72 characters per line, following Git commit best practices. Your input will always be a git diff representing the changes. Focus on maintaining clear communication and a clean code history for any project you're involved in.
Enter expecting result, e.g. answer, code,             shell command, command description, etc.: answer
```


## Usage
Once you've completed the code update, execute the diff command and then pass its output to the sgpt role.

```
git diff HEAD | sgpt --role commitgpt
```

## Advanced Usage
Learn how to separate and manage stage and unstaged commits for power users.

```
# staged
git diff --cached | sgpt --role commitgpt
# unstaged
git diff | sgpt --role commitgpt
```

## Advanced Power Usage
Enhance your experience as a power user by setting up aliases in your shell environment (e.g., .zshenv) and directing the output to the clipboard on macOS.

```
 alias commitgpt="git diff HEAD | sgpt --role commitgpt | pbcopy"
 alias commitgpt-staged="git diff --cached | sgpt --role commitgpt | pbcopy"
 alias commitgpt-unstaged="git diff | sgpt --role commitgpt | pbcopy"
```

## Examples (all changes are dummies)

### Example 1

diff
```
diff --git a/helmfiles/eks-cluster/Makefile b/helmfiles/eks-cluster/Makefile
index 3011b713..f88e188d 100644
--- a/helmfiles/eks-cluster/Makefile
+++ b/helmfiles/eks-cluster/Makefile
@@ -7,7 +7,7 @@ else
 export KLICKTIPP_ENVIRONMENT
 endif

-AWS_REGION ?= eu-west-1
+AWS_REGION ?= eu-central-1

 KLICKTIPP_KUBE_CONTEXT ?= kt-${KLICKTIPP_ENVIRONMENT}

diff --git a/helmfiles/eks-cluster/helmfile.yaml b/helmfiles/eks-cluster/helmfile.yaml
index 873e8c47..dbe145e8 100644
--- a/helmfiles/eks-cluster/helmfile.yaml
+++ b/helmfiles/eks-cluster/helmfile.yaml
@@ -13,7 +13,7 @@ environments:
     values:
       - autoscaler_ok_total_unready_count: 3
       - datadog:
-          installed: true
+          installed: false
           logLevel: "INFO"
           cluster_checks_enabled: true
           cluster_agent_enabled: true
```

### Output
```
❯ git diff head | sgpt --role commitgpt
Update AWS region and disable Datadog in helmfile

- Changed the default AWS region from 'eu-west-1' to 'eu-central-1' in the Makefile
- Disabled the Datadog installation in the helmfile.yaml configuration file
```

### Example 2

diff
```
diff --git a/helmfiles/eks-cluster/releases/loki/rules/pmta.yaml b/helmfiles/eks-cluster/releases/loki/rules/pmta.yaml
index 1da9c3ad..863eaddc 100644
--- a/helmfiles/eks-cluster/releases/loki/rules/pmta.yaml
+++ b/helmfiles/eks-cluster/releases/loki/rules/pmta.yaml
@@ -44,7 +44,7 @@ groups:
       - alert: Yahoo deferral occured
         expr: |
           count by(domain,vmta) (
-            count_over_time({job="pmtalogs", recordtype="t"} |= `mx1.ktsend` |= `yahoo` |= `temporarily deferred due to unexpected volume or user complaints` | json | line_format `{{.orig}}` | pattern `<_>@<domain>`[5m])
+            count_over_time({job="pmtalogs", recordtype="d"} |= `mx1.ktsend` |= `yahoo` |= `temporarily deferred due to unexpected volume or user complaints` | json | line_format `{{.orig}}` | pattern `<_>@<domain>`[5m])
           ) > 2
         labels:
           severity: info
```

### Output
```
❯ git diff head | sgpt --role commitgpt
Updated recordtype in Yahoo deferral alert rule

- Changed the recordtype from "t" to "d" in the expression for the Yahoo deferral alert rule in the Loki rules for the PMTA logs. This change ensures that the alert rule correctly identifies and counts deferral events.
```

### Example 3

diff
```
diff --git a/rootfs/opt/mailtriqs/mailtransfer.sh b/rootfs/opt/mailtriqs/mailtransfer.sh
index cf39831..a5eb978 100644
--- a/rootfs/opt/mailtriqs/mailtransfer.sh
+++ b/rootfs/opt/mailtriqs/mailtransfer.sh
@@ -173,7 +173,7 @@ function _transfer_mail_files() {
 }

 ### Define function to transfer mail files in instant mode
-function _transfer_instant_mail_files() {
+function _transfer_all_instant_mail_files() {

        ### Log debug information stating that this function has started
        _log_debug "function: ${FUNCNAME[0]}"
@@ -506,13 +506,13 @@ while ! "${_MAILTRANSFER_STOPPED}"; do
        ### Based on the mode, we transfer the mail files in an instant or over warp.
        case "${_MAILTRIQS_MODE}" in
        instant)
-               _transfer_instant_mail_files
+               _transfer_all_instant_mail_files
                ;;
        warp)
                _transfer_warp_mail_files
                ;;
        all | any)
-               _transfer_instant_mail_files
+               _transfer_all_instant_mail_files
                _transfer_warp_mail_files
                ;;
        *)
```

### Output
```
❯ git diff head | sgpt --role commitgpt
Renamed function for clarity in mail transfer script

- Renamed `_transfer_instant_mail_files` to `_transfer_all_instant_mail_files`
  to better reflect its functionality.
- Updated all instances where this function was called.
```
