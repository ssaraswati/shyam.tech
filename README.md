# shyam.tech

personal site based on  [Jekyll-Uno]((https://github.com/joshgerdes/jekyll-uno) - a minimal, responsive theme for Jekyll based on the [Uno]((https://github.com/joshgerdes/jekyll-uno) theme for Ghost.

# Hosting
Hosted out of s3 with cloudfront providing ssl

Template is reusable for other static websites such as spa apps (Vue, React, Angular ect) by providing an appropriate buildspec.yaml
It will create 2 buckets (Build and Site) with website hosting enabled on the Site bucket and a cloudfront distribution setup with cdn-, www and @ domain alias
Build and deploy using CodeBuild

### Copyright and license

It is under [the MIT license](/LICENSE).