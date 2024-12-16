# Mafyuh.com Source Code

## Workflow Pipeline
### High level
Posts start in [Obsidian](https://obsidian.md/), where I can just copy and paste images directly into posts and those will automatically translate once I'm ready to push to Github. Run a [script](https://github.com/Mafyuh/mafyuh.github.io/blob/main/obsidian-hugo.ps1) which I have binded to a [Autohotkey](https://www.autohotkey.com/) hotkey which pushes changes made in Obsidian to Git. Git then runs [this workflow](https://github.com/Mafyuh/mafyuh.github.io/blob/main/.github/workflows/hugo.yml) and posts to Github pages, where Cloudflare has a CNAME for `mafyuh.github.io` of `mafyuh.com`

This setup was inspired by NetworkChuck in [this video](https://www.youtube.com/watch?v=dnE7c0ELEH8) and scripts are just modifications of [his](https://blog.networkchuck.com/posts/my-insane-blog-pipeline/). Would watch his video and read his blog post for missing information.





