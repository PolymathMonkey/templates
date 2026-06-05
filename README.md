# Threat Hunt Template
 
Org-mode capture template for structured threat hunt documentation.
Queries run directly against Elasticsearch via org-babel.
 
Background and methodology: [Threat Hunting like a Nerd](https://polymathmonkey.github.io/weblog/artifacts/threathuntinglikeanerd/)
 
> This workflow is documented in detail on the blog:
> [Threat Hunting like a nerd](https://polymathmonkey.github.io/weblog/artifacts/threathuntinglikeanerd/)
 
## Setup
 
### 1. Place the template
 
```bash
cp threathunt-template.org ~/.emacs.d/templates/threathunt-template.org
```
 
### 2. Add to Emacs config
 
```elisp
(setq org-capture-templates
  '(("t" "Threat Hunt" plain
     (file (lambda ()
       (let ((name (read-string "Hunt name: ")))
         (expand-file-name
           (concat "hunts/TH-"
                   (format-time-string "%Y%m%d")
                   "-" name ".org")
           org-directory))))
     (file "~/.emacs.d/templates/threathunt-template.org"))))
```
 
Set `org-directory` to wherever your hunt files should live, e.g.:
 
```elisp
(setq org-directory "~/org/hunts")
```
 
### 3. Configure ES credentials
 
The template uses `elastic:PASSWORD` as placeholder. Replace globally before
running queries, or set a file-local variable at the bottom of each hunt file:
 
```org
# Local Variables:
# eval: (setq-local es-password "yourpassword")
# org-confirm-babel-evaluate: nil
# End:
```
 
Or use a shell alias on siem1 so the password never lands in the file:
 
```bash
# ~/.bashrc on siem1
alias escurl='curl -s -k -u elastic:$(cat ~/.es_password)'
```
 
Then replace `curl -s -k -u elastic:PASSWORD` with `escurl` in the src blocks.
 
## Usage
 
`M-x org-capture` → `t` → enter hunt name (e.g. `ssh-bruteforce-cn`) → RET
 
Creates `~/org/hunts/TH-20260605-ssh-bruteforce-cn.org` pre-filled with the
template. Fill in Hypothesis and Trigger, then run queries with `C-c C-c` on
each src block.
 
## Running queries
 
Queries execute over SSH on siem1 via the header:
 
```org
#+PROPERTY: header-args:shell :dir /ssh:siem1:
```
 
Requires TRAMP SSH access to siem1. Test with:
 
```
M-x shell-command RET /ssh:siem1:hostname RET
```
 
## Validating queries before publish
 
```bash
./validate_hunt_template.sh <es_password>
```
 
All queries must exit 0 before publishing a hunt as a blog post.
 
## Publishing to blog
 
1. Copy finished hunt content into `content-org/securityresearch.org` as a new heading
2. Set status to `DONE`
3. `C-c C-e H H` to export via ox-hugo
4. Commit `content/securityresearch/<slug>.md`
