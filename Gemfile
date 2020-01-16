source 'https://rubygems.org'

gem "jekyll", "~> 4.0.0"

gem "minima", "~> 2.5"

group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jemoji"
  gem "jekyll-paginate"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :install_if => Gem.win_platform?
