# frozen_string_literal: true

require 'rake/clean'
require 'erb'

JRUBY_VERSION = '9.2.5.0'
MRI_RUBY = '2.3.0'
PRIORITY = '51'
TAR_BALL = "jruby-dist-#{JRUBY_VERSION}-bin.tar.gz"
TAR_BALL_SHA = [TAR_BALL, 'sha256'].join('.')

set_gem_home = ERB.new <<~GEMPATH
  # Set GEM_HOME and add GEM_HOME/bin to PATH
  export GEM_HOME="$HOME/.gem/ruby/<%= MRI_RUBY %>/"
  export GEM_PATH="$HOME/.gem/ruby/<%= MRI_RUBY %>/"
  export PATH="$GEM_HOME/bin:$PATH"

GEMPATH

WARNING = <<~WARN
  WARNING: Something has gone wrong with download
  check that the JRUBY_VERSION and download path
  are still valid

WARN

URL = [
  'https://repo1.maven.org',
  'maven2',
  'org',
  'jruby',
  'jruby-dist',
  JRUBY_VERSION,
  TAR_BALL
].join('/')

SHA256 = [
  URL,
  'sha256'
].join('.')

CLOBBER << TAR_BALL
CLOBBER << TAR_BALL_SHA

task default: %i[install_jruby gem_home install_gem install_samples]

file TAR_BALL do
  begin
    sh "wget #{URL}"
    sh "wget #{SHA256}"
  rescue
    warn(WARNING)
  end
  value = File.read(TAR_BALL_SHA)
  check_sha256(TAR_BALL, value)
end

desc 'Install JRuby'
task install_jruby: TAR_BALL do
  tar_ball = File.join(Dir.pwd, TAR_BALL)
  Dir.chdir('/opt') do
    sh "sudo tar xzvf #{tar_ball}"
  end
  sh "sudo update-alternatives --install /usr/bin/jruby jruby /opt/jruby-#{JRUBY_VERSION}/bin/jruby #{PRIORITY}"
  sh "sudo update-alternatives --install /usr/bin/jirb jirb /opt/jruby-#{JRUBY_VERSION}/bin/jirb #{PRIORITY}"
  sh "sudo update-alternatives --install /usr/bin/jgem jgem /opt/jruby-#{JRUBY_VERSION}/bin/jgem #{PRIORITY}"
end

def check_sha256(filename, expected_hash)
  require 'digest'
  sha256 = Digest::SHA256.new
  File.open(filename, 'r') do |f|
    while (buf = f.read(4096))
      sha256.update(buf)
    end
  end
  next if sha256.hexdigest == expected_hash

  raise "bad sha256 checksum for #{filename} (expected #{expected_hash} got #{sha256.hexdigest})"
end

desc 'check for GEM_HOME'
task :gem_home do
  puts "check gem home"	
  next unless ENV['GEM_HOME'].nil?

  puts 'setting local GEM_HOME and PATH'
  gem_dir = File.join(ENV['HOME'], ".gem/ruby/#{MRI_RUBY}")
  sh "mkdir -p #{gem_dir}" unless Dir.exist?(gem_dir)
  profile = File.join(ENV['HOME'], '.profile')
  File.open(profile, 'a') do |file|
    file << set_gem_home.result(binding)
  end
  # set GEM_HOME so we can use it in ruby script to install picrate
  ENV['GEM_HOME'] ||= gem_dir
end

desc 'install gem'
task install_gem: :gem_home do
  sh 'jgem install picrate'
end

desc 'install samples'
task install_samples: :install_gem do
  picrate_bin = File.join(ENV['GEM_HOME'], 'bin', 'picrate')
  sh "#{picrate_bin} --install Samples"
end
