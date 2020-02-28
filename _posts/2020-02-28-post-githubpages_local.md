---
title: "Github Pages 로컬에서 돌리는 법 (Ubuntu)"
date: 2020-02-28
categories:
  - blog
tags:
  - web
  - github
---

### [Jekyll install][jekyllinstall]
~~~
sudo apt-get install ruby-full build-essential zlib1g-dev

# gem을 sudo로 까는 것을 피하기 위한 조치 
echo '# Install Ruby Gems to ~/gems' >> ~/.bashrc
echo 'export GEM_HOME="$HOME/gems"' >> ~/.bashrc
echo 'export PATH="$HOME/gems/bin:$PATH"' >> ~/.bashrc
source ~/.bashrc

# Jekyll install
gem install jekyll bundler
~~~

### [실행 방법][localhowto]
1. 원하는 github page를 git clone
2. clone한 directory root에서 bundle install 하여 설치
3. bundle exec jekyll serve
4. 127.0.0.1:4000 접속해서 보기

[jekyllinstall]: https://jekyllrb.com/docs/installation/ubuntu/
[localhowto]:https://help.github.com/en/github/working-with-github-pages/testing-your-github-pages-site-locally-with-jekyll