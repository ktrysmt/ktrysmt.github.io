# Kotaro Yoshimatsu

* [Twitter](https://twitter.com/ktrysmt)
* [Github](https://github.com/ktrysmt)
* [Blog (ja)](https://ktrysmt.github.io/blog/)
* kotaro.yoshimatsu@gmail.com

in local,

```
mise install ruby 3.1
mise use ruby@3.1
gem install jekyll jekyll-seo-tag jemoji jekyll-mentions jekyll-redirect-from jekyll-sitemap jekyll-feed -N
jekyll s --plugins $(fd -t directory . $(mise where ruby) | grep gems/jekyll-seo-tag | head -1)
```
