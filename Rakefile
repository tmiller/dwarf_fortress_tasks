require 'rubygems'
require 'bundler'
Bundler.require
require 'rake/clean'

CLEAN.include('df', '*.tar.bz2', '*.tar')
CLOBBER.include('df')

TILESET = 'vidumec-12x12.png'
COLOR_SCHEME = 'another_natural_scheme.txt'

module DwarfConfig
  def config(settings={})
    settings.each do |k,v|
      v = if v.class == Array
            v.join(':')
          else
            v.to_s.sub('true', 'yes').sub('false', 'no')
          end
      v = v.upcase unless v == TILESET
      sub! /^\[(#{k.to_s.upcase}):[^\]]+\]/, "[\\1:#{v}]"
    end
  end
end

def read_config(filename)
  File.read(filename).extend DwarfConfig
end

def write_config(filename, settings={})
  puts "Configuring #{filename}"
  file = read_config "df/data/init/#{filename}"
  file.config settings
  File.open("df/data/init/#{filename}", 'w') do |f|
    f << file
  end
end

task :default => [:config, :setup]

task :config => 'df' do

  write_config 'init.txt', {
    intro: false, 
    sound: false,
    windowedx: 120,
    windowedy: 71,
    print_mode: 'standard',
    font: TILESET
  }

  write_config 'd_init.txt', {
    embark_rectangle: [6, 6],
    coffin_no_pets_default: true,
    idlers: 'top',
    population_cap: 100,
    baby_child_cap: [20, 20],
    varied_ground_tiles: false,
    show_flow_amounts: true
  }
end

task :setup => 'df' do
  cp "colors/#{COLOR_SCHEME}", 'df/data/init/colors.txt'
  cp "tilesets/#{TILESET}", "df/data/art/#{TILESET}"
end

file 'df' => ['df.tar'] do |t|
  sh "tar -xf #{t.prerequisites.first}"
  mv FileList['df_{osx,linux}'].first, t.name
end

file 'df.tar' => ['df.tar.bz2'] do |t|
  sh "bunzip2 #{t.prerequisites.first}"
end

file 'df.tar.bz2' do
  url = 'http://bay12games.com/dwarves/'
  link_text = case
              when OS.mac? then 'Mac (Intel)'
              when OS.linux? then 'Linux'
              end
  filename = Mechanize.new.get(url).link_with(text: link_text).href
  sh "curl -# #{url}#{filename} -o df.tar.bz2" unless filename.empty?
end

task :install => [:config, :setup] do
  mv 'df', "#{ENV['HOME']}/df"
end
