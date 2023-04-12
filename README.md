# Kotaro Yoshimatsu

* [Twitter](https://twitter.com/ktrysmt)
* [Github](https://github.com/ktrysmt)
* [Blog (ja)](https://ktrysmt.github.io/blog/)
* kotaro.yoshimatsu@gmail.com

in local,

```
asdf install ruby 2.7.7
asdf global ruby 2.7.7
gem install jekyll jekyll-seo-tag jemoji jekyll-mentions jekyll-redirect-from jekyll-sitemap jekyll-feed -N
jekyll s --plugins $(fd -t directory . $(asdf where ruby) | grep gems/jekyll-seo-tag | head -1)
```
