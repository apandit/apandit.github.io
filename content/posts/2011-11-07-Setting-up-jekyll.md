---
date: 2011-11-07T00:00:00Z
title: Setting up Jekyll on Win7
---

Jekyll takes a bit of effort to set up on Win7. I was hoping to just grab Ruby and gems on Cygwin to install it but that didn't work out as planned. So here is how I made Jekyll work on Win7 64-bit.

## Installing Ruby

Grab __Ruby Installer__ from their [downloads](http://rubyinstaller.org/downloads/) page. I used [Ruby 1.9.3-p0](http://rubyforge.org/frs/download.php/75465/rubyinstaller-1.9.3-p0.exe). The install should go by smoothly.

### Installing dev-kit

From the same downloads page above, download the development kit. I used [DevKit-tdm-32-4.5.2-20110712-1620-sfx](http://github.com/downloads/oneclick/rubyinstaller/DevKit-tdm-32-4.5.2-20110712-1620-sfx.exe)

This is just a 7z file so extract it to a folder (%DEVKITFOLDER%). Open a cmd window and @cd@ to this folder.

      ruby dk.rb init

Confirm that the __config.yml__ contains your ruby path.

      ruby dk.rb install

### Installing Jekyll

      gem install jekyll

#### Modifying Redcloth to work (only on 4.2.8)

Find __redcloth.rb__ . It should be in a folder similar to __C:\Ruby193\lib\ruby\gems\1.9.1\gems\RedCloth-4.2.8\lib__.

On line 10, replace with `prefix = ''` This will allow it to find `redcloth_scan.so which is in the same folder.

#### (Optional) Modifying spawn to not use Posix

Find __albino.rb__. You'll need to patch it using [this fix](https://gist.github.com/1185645).

In case that page goes missing: 

      diff --git a/lib/albino.rb b/lib/albino.rb<br />
      index 387c8e9..b77d55e 100644<br />
      --- a/lib/albino.rb<br />
      +++ b/lib/albino.rb<br />
      @@ -1,4 +1,5 @@<br />
      require 'posix-spawn'<br />
      +require 'rbconfig'<br />
      <br />
      <br />
      =begin Wrapper for the Pygments command line tool, pygmentize. =end<br />
      @@ -84,11 +85,21 @@ class Albino<br />
        proc_options":timeout" = options.delete(:timeout) || self.class.timeout_threshold<br />
        command = convert_options(options)<br />
        command.unshift(bin)<br />
      -    Child.new(*(command + "proc_options.merge(:input => write_target)"))<br />
      +    if RbConfig::CONFIG"'host_os'" =~ /(mingw|mswin)/<br />
      +      output = ''<br />
      +      IO.popen(command, mode='r+') do |p|<br />
      +        p.write @target<br />
      +        p.close_write<br />
      +        output = p.read.strip<br />
      +      end<br />
      +      output<br />
      +    else<br />
      +      Child.new(*(command + "proc_options.merge(:input => write_target)"))<br />
      +    end<br />
        end<br />
      <br />
        def colorize(options = {})<br />
      -    out = execute(options).out<br />
      +    out = RbConfig::CONFIG"'host_os'" =~ /(mingw|mswin)/ ? execute(options) : execute(options).out
