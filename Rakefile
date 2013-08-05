ENV['RACK_ENV'] = "development"

require "bundler/gem_tasks"
require "rspec/core/rake_task"

RSpec::Core::RakeTask.new(:spec)

task :default => :spec

task :pry do
  require 'pry'; binding.pry
end

namespace :assets do
  task :precompile do
    public = File.join File.dirname(__FILE__), %w(lib bait public)
    require 'bait/api'
    include Sinatra::AssetSnack::InstanceMethods
    Sinatra::AssetSnack.assets.each do |assets|
      compiled_path = File.join public, assets[:route]
      puts "compiling #{compiled_path}"
      File.open(compiled_path, 'w') do |file|
        response = compile assets[:paths]
        file.write response[:body]
      end
    end
  end
end


def git_master?
  `git branch | grep '* master'`
  $?.exitstatus == 0
end

def git_dirty?
  `git status --porcelain`.match(/^\sM/)
end

namespace :gem do
  task :build => 'assets:precompile' do
    `bundle install`
    if git_dirty?
      puts "dirty! commit first before building"
    else
      if git_master?
        puts "On master branch"
        `rspec spec`
        if $?.exitstatus == 0
          puts "Specs pass. you're ready"
          puts `gem build bait.gemspec`
          puts "Done! You can gem push that now"
        else
          puts "Uhh.. you have failing specs -- not building the gem"
        end
      else
        puts "I'll only build the gem on the master branch"
      end
    end
  end
end
