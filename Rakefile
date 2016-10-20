autoload :FileUtils, 'fileutils'
autoload :JSON, 'json'

PACKAGE_JSON = Class.new(Hash) do
  FILENAME = 'package.json'

  def initialize
    self.merge! JSON.load(open FILENAME)
  end

  alias old_update []=
  def []=(k, v)
    old_update(k, v)
    write
    v
  end

private
  def write
    File.write(FILENAME, JSON.pretty_generate(self))
  end
end.new

def version=(v)
  PACKAGE_JSON["version"] = v
end

def version
  PACKAGE_JSON["version"]
end

def ver_bump(idx, type='')
  $v = self.version = version.split('.').map do |p|
    p = Integer(p)+1 if idx == 0
    idx -= 1
    p
  end.join('.')
  sh <<-CMD
  git add package.json && \
  git commit -sS -m '#{type}version bump to #{$v}' && \
  git tag -s '#{$v}' -m '#{$v}'
  CMD
end

# ---------- tasks

desc 'publish atom package using --tag'
task :publish, [:tag] do |_, args|
  sh "apm publish --tag #{args[:tag]}"
end

desc 'package.json version'
task :version do
  puts version
end

desc 'package.json major version only'
task 'version:major' do
  puts version.split('.')[0]
end

desc 'package.json minor version only'
task 'version:minor' do
  puts version.split('.')[1]
end

desc 'package.json patch version only'
task 'version:patch' do
  puts version.split('.')[2]
end

desc 'bump patch version'
task 'version:bump' do
  ver_bump 2
end

desc 'bump major version'
task 'version:major:bump' do
  ver_bump 0, 'major '
end

desc 'bump minor version'
task 'version:minor:bump' do
  ver_bump 1, 'minor'
end

desc 'bump & publish a new patch version'
task 'release' => ['version:bump', 'git:push'] do
  Rake::Task[:publish].invoke($v)
end

desc 'bump & publish a new minor version'
task 'release:minor' => ['version:major:bump', 'git:push'] do
  Rake::Task[:publish].invoke($v)
end

desc 'bump & publish a new major version'
task 'release:major' => ['version:major:bump', 'git:push'] do
  Rake::Task[:publish].invoke($v)
end

desc 'setup development environment'
task :setup => %w[setup:less setup:git-hooks]

desc 'setup only Less'
task 'setup:less' do
  sh 'npm i -g less'
end

task 'git:push' do
  sh 'git push --mirror'
end

desc 'setup only git-hooks'
task 'setup:git-hooks' do
  if File.directory? 'git-hooks'
    Dir['git-hooks/*'].each do |hook|
      File.chmod 0755, hook
      FileUtils.ln_sf "../../#{hook}", '.git/hooks/'
    end
  end
end
