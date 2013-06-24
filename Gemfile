source "http://rubygems.org"

gem "eventmachine"
gem "em-http-request"
gem "nats"
gem "ruby-hmac"
gem "uuidtools"
gem "datamapper", "= 1.1.0"
gem "dm-sqlite-adapter"
gem "do_sqlite3"
gem "sinatra", "~> 1.2.3"
gem "thin"


gem 'vcap_common', :path => '../../common', :require => ['vcap/common', 'vcap/component']
gem 'vcap_logging', '>=0.1.3', :require => ['vcap/logging']
gem "vcap_services_base", :path => "../base"

group :test do
  gem "rake"
  gem "rspec"
  gem "simplecov"
  gem "simplecov-rcov"
  gem "ci_reporter"
end
