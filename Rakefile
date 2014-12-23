require 'open3'
require 'ostruct'
require 'singleton'

#
# Constants
#
VERBOSE = true
NOOP = false

#
# Extended String
#
class String
  def red;     "\033[31m#{self}\033[0m"; end
  def green;   "\033[32m#{self}\033[0m"; end
  def yellow;  "\033[33m#{self}\033[0m"; end
  def blue;    "\033[34m#{self}\033[0m"; end
  def magenta; "\033[35m#{self}\033[0m"; end
  def cyan;    "\033[36m#{self}\033[0m"; end
  def gray;    "\033[37m#{self}\033[0m"; end
end

class Vars
  include Singleton

  attr_reader :bin, :plist

  def initialize
    # bin
    @bin = OpenStruct.new({
      :src => File.join(__dir__, 'bin/caplog'),
      :dst => File.expand_path('~/.bin/caplog')
    })

    # plist
    parts = %x(hostname -f).strip.split('.')
    parts = parts.push('localhost') if parts.length <= 0
    parts = parts.push('localdomain') if parts.length <= 1

    attrs = OpenStruct.new({
      :label    => parts.reverse.push('caplog').join('.'),
      :program  => File.expand_path('~/.bin/caplog'),
      :username => %x(whoami).strip
    })

    @plist = OpenStruct.new({
      :label   => attrs.label,
      :path    => File.join(File.expand_path('~/Library/LaunchAgents'), attrs.label + '.plist'),
      :content => File.read(File.join(__dir__, 'conf/domain.host.caplog.plist.tmpl')) % attrs.to_h
    })
  end
end

#
# Helper functions
#
class Helper

  #
  # copy file
  #
  def self.cp(src, dst)
    return self.exec("cp #{src} #{dst}")
  end

  #
  # remove file
  #
  def self.rm(file)
    return self.exec("rm #{file}")
  end

  #
  # load launchctl file
  #
  def self.load(file)
    return self.exec("launchctl load #{file}")
  end

  #
  # unload launchctl file
  #
  def self.unload(file)
    return self.exec("launchctl unload #{file}")
  end

  #
  # detect launchctl loaded
  #
  def self.loaded?(label)
    return Open3.capture3("launchctl list #{label}").last.success?
  end

  #
  # exec shell command
  #
  def self.exec(command, verbose: VERBOSE, noop: NOOP)
    puts "[EXEC] #{command}".green if verbose

    return true if noop

    Process.waitpid(Process.spawn(command))

    return $?.success?
  end
end

#
# Actions
#
class Action

  #
  # install
  #
  def self.install
    vars = Vars.instance

    # gem
    Helper.exec('bundle install')

    puts if VERBOSE

    # bin
    if File.executable?(vars.bin.dst)
      puts "[WARN] File already exists: #{vars.bin.dst}".yellow
    else
      Helper.cp(vars.bin.src, vars.bin.dst)
    end

    # plist
    if File.exists?(vars.plist.path)
      puts "[WARN] File already exists: #{vars.plist.path}".yellow
    else
      puts "[INFO] create file #{vars.plist.path}".cyan
      File.write(vars.plist.path, vars.plist.content)
    end

    # load
    if Helper.loaded?(vars.plist.label)
      puts "[WARN] Label already loaded: #{vars.plist.label}".yellow
    else
      Helper.load(vars.plist.path)
    end
  end

  #
  # uninstall
  #
  def self.uninstall
    vars = Vars.instance

    # unload
    if !Helper.loaded?(vars.plist.label)
      puts "[WARN] Label not loaded: #{vars.plist.label}".yellow
    else
      Helper.unload(vars.plist.path)
    end

    # plist
    if !File.exists?(vars.plist.path)
      puts "[WARN] File not found: #{vars.plist.path}".yellow
    else
      Helper.rm(vars.plist.path)
    end

    # bin
    if !File.executable?(vars.bin.dst)
      puts "[WARN] File not found: #{vars.bin.dst}".yellow
    else
      Helper.rm(vars.bin.dst)
    end
  end

  #
  # show status
  def self.status
    vars = Vars.instance

    # bin
    if File.executable?(vars.bin.dst)
      puts "File found: #{vars.bin.dst}".green
    else
      puts "File not found: #{vars.bin.dst}".red
    end

    # plist
    if File.exists?(vars.plist.path)
      puts "File found: #{vars.plist.path}".green
    else
      puts "File not found: #{vars.plist.path}".red
    end

    # load
    if Helper.loaded?(vars.plist.label)
      puts "Label loaded: #{vars.plist.label}".green
    else
      puts "Label not loaded: #{vars.plist.label}".red
    end
  end
end

#
# Tasks
#
desc 'install'
task :install do
  Action.install
end

desc 'uninstall'
task :uninstall do
  Action.uninstall
end

desc 'status'
task :status do
  Action.status
end

# vim: ft=ruby sw=2 ts=2 sts=2
