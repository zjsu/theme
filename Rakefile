require 'erb'
require 'ostruct'
require 'tmpdir'

task :default => :setup

def get_option(prompt, default)
  print "#{prompt}[#{default}]: "
  s = STDIN.gets.chomp
  return s.empty? ? default : s
end

def ask_options
  options = {
    dest: '.',
    branch: 'gh-pages',
    repo: 'zjsu/theme',
    title: 'ZJSU Theme',
    baseurl: '/theme',
    disqus: 'zjsutheme',
  }

  options[:dest] = get_option('Enter project folder', options[:dest])
  options[:dest] = File.expand_path(options[:dest])

  options[:branch] = get_option('Enter branch', options[:branch])
  options[:repo] = get_option('Enter repo', options[:repo])
  options[:title] = get_option('Enter title', options[:title])
  options[:baseurl] = get_option('Base URL', options[:baseurl])
  options[:disqus] = get_option('Disqus short name', options[:disqus])

  return options
end

def git?
  return system('git rev-parse >/dev/null 2>&1')
end

def clean?
  return system('git status --porcelain >/dev/null 2>&1')
end

def precheck(dir)
  Dir.chdir(dir) do
    unless git?
      abort "Error: #{dir} is not under git control"
    end

    unless clean?
      abort "Error: #{dir} is not clean"
    end
  end
end

def copy_site(src, dest)
  unless File.exist?("#{dest}/index.md")
    cp "#{src}/site/index.md", dest
  end

  unless File.exist?("#{dest}/_data")
    cp_r "#{src}/site/_data", dest
  end
end

def deploy(options)
  pwd = Dir.pwd  
  Dir.mktmpdir do |dir|
    cp_r 'template/.', dir
    Dir.glob("#{dir}/**/*.erb", File::FNM_DOTMATCH) do |f|
      out = File.join(File.dirname(f), File.basename(f, '.erb'))
      erb = ERB.new(File.read(f))
      File.open(out, 'w+') do |f|
        f.write(erb.result(OpenStruct.new(options).instance_eval { binding }))
      end
      rm_f f
    end

    Dir.chdir(options[:dest]) do
      unless system("git checkout #{options[:branch]}")
        abort "No branch #{options[:branch]}"
      end
      entries = Dir.entries(dir).reject{|f| f == '.' || f == '..' }
      rm_rf entries
      cp_r "#{dir}/.", '.'
      copy_site(pwd, options[:dest])
      system("git add . && git commit -am 'Theme update'")
    end
  end
end

task :setup do
  options = ask_options

  precheck(options[:dest])
  deploy(options)
end
