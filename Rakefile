require 'net/https'
require 'json'

task :default do
  token = ENV['TOKEN']
  slug  = Time.now.utc.strftime "%Y-%m-%d-%H-%M-%S-%L"

  # Check token
  fail "TOKEN not set" if token.nil? or token.empty?

  # Create Branch
  sh "git clone https://#{token}:x-oauth-basic@github.com/travis-repos/test-repo-2.git tmp-#{slug}"
  cd "tmp-#{slug}"
  sh "git checkout -b #{slug}"
  File.write(slug, slug)
  sh "git add #{slug}"
  sh "git commit -m #{slug}"
  sh "git push origin #{slug}"
  sh "git checkout master"

  # GitHub API
  http         = Net::HTTP.new('api.github.com', 443)
  http.use_ssl = true
  headers      = { "Authorization" => "token #{token}" }

  # Create Pull Request
  payload      = JSON.dump(title: slug, base: 'master', head: slug)
  response     = http.post("/repos/travis-repos/test-repo-2/pulls", payload, headers)
  pull_request = JSON.load(response.body)
  state        = pull_request["mergeable_state"]

  # Wait for merge commit
  puts "waiting for github, press CTRL-C to give up"
  while %w[checking unknown].include? state
    print "."
    sleep 1
    response = http.get("/repos/travis-repos/test-repo-2/pulls/#{pull_request['number']}", headers)
    state    =  JSON.load(response.body)["mergeable_state"]
  end
  puts " => #{state}"

  # Remove traces
  cd ".."
  rm_rf "tmp-#{slug}"
end
