# frozen_string_literal: true

source "https://rubygems.org"

# Ruby 3.4.10 constraint — exercises the bundled PM change probe.
# Ruby 3.4 extracted several stdlib gems (bigdecimal, mutex_m,
# ostruct) so they must now be declared as explicit Bundler deps.
# The pessimistic ~> 3.4.10 pin constrains the resolver to the
# 3.4.10 patch-level, stress-testing Bundler's version negotiation.
ruby "~> 3.4.10"

# ---- stdlib-extracted gems (Ruby 3.4 bundled PM change) ----------
# bigdecimal was extracted from stdlib in Ruby 3.4. Bundler must
# resolve it as a real gem; Mend must detect it as a registry dep.
gem "bigdecimal", "~> 3.1"

# mutex_m extracted from stdlib in Ruby 3.4.
gem "mutex_m", "~> 0.3"

# ostruct extracted from stdlib in Ruby 3.4.
gem "ostruct", "~> 0.6"

# ---- core gems with complex transitive graphs --------------------
# rack 3.1 pulls rack-session and rack-utils; exercises transitive
# detection under the Ruby 3.4 resolver path.
gem "rack", "~> 3.1"

# faraday 2.x has a large dependency fan-out (faraday-net_http,
# net-http, uri) and historically relied on stdlib net/* gems that
# are now explicit Bundler deps in Ruby 3.4.
gem "faraday", "~> 2.12"

# ---- development group -----------------------------------------
group :development do
  gem "rake", "~> 13.2"
end

group :test do
  gem "minitest", "~> 5.25"
end
