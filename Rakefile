#!/usr/bin/env ruby
require 'pathname'
require 'erb'
require 'yaml'
require 'yaml/dbm'
require 'json'
require 'pp'
require 'digest/sha1'

if File.exists?('config.yml')
  config = YAML.load_file(File.expand_path 'config.yml')
  config['digest_pages'] = Digest::SHA1.hexdigest(`cat pages/*`)
  config['digest_git_sha'] = `git log | head -n1 | cut -d ' ' -f 2`.chomp
  config['filename'] = "Contract-#{Time.now.strftime('%Y-%m-%d-%H-%M-%S')}"
  config['date'] = Time.now.strftime('%Y-%m-%d-%H-%M-%S')
else
  puts "Run `mv config.yml.example config.yml`"
  puts "Then fill in values as needed in config.yml"
  exit(1)
end


def compile_pdf(config)
  config = config
  contract_output_name = "Contract-#{Time.now.strftime('%Y-%m-%d-%H-%M-%S')}"

  pre_hook(contract_output_name)

  files = ""

  Dir.glob("./pages/*.md*").each do |file|
    file = Pathname.new(file)
    basename = file.basename.to_s
    # if file.include? ".md"
      file_markdown = basename.gsub(/.erb$/i, '')
      file_html = basename.gsub(/.md.erb$/i, '.html')

      process_erb(basename, file_markdown, config)
      system "markdown 'markdown/#{file_markdown}' > 'html/#{file_html}'"
      files << "'html/#{file_html}' " unless basename == '00-Title_Page.md.erb'
    # end
  end

  html_doc =  "htmldoc --book --titlefile 'html/00-Title_Page.html'"
  html_doc += " -f #{ contract_output_name }.pdf #{files}"

  system html_doc

  system "shasum #{contract_output_name}.pdf > #{contract_output_name}.sha1"
  config['digest_pdf'] = `shasum #{contract_output_name}.pdf`.chomp.split(" ")[0]

  pp @db[config['digest_pdf']] = config

  File.open(File.expand_path('log/contract.log'), 'a' ) do |f|
    entry = config['digest_pdf']
    entry << ": "
    entry << config.to_json
    entry << "\n"
    f.write entry
  end
ensure
  post_hook
end

def process_erb(basename=basename, file_markdown=file_markdown, config)
  rendered_template = ERB.new(File.read("pages/#{basename}")).result(config.send(:binding))
  File.write("markdown/#{file_markdown}", rendered_template)
end

def pre_hook(contract_output_name)
  check_for_dependencies
  contract_output_name = contract_output_name
  system "rm *.pdf &2> /dev/null"
  system "rm *.sha1 &2> /dev/null"
  %w[html markdown log].each do |dir|
    FileUtils.mkdir "#{dir}" unless Dir.exists?(dir)
  end
  FileUtils.touch('log/contract.log') unless File.exists?('log/contract.log')
  establish_database_connection
end

def establish_database_connection
  return if @db
  db_filename = File.expand_path('log/contract.yaml_dbm')
  if File.exists?(db_filename + ".db")
    @db = YAML::DBM.open(db_filename)
  else
    @db = YAML::DBM.new(db_filename)
  end
end

def check_db_record
  establish_database_connection
  sha = ENV['SHA'] || @db.keys.last
  pp @db[sha]
end

def check_for_dependencies
  %w[shasum markdown htmldoc].each do |tool|
    unless `which #{tool}`
      puts "Please install #{tool}"
      exit(1)
    end
  end
end
def post_hook
  %w[html markdown].each do |dir|
    system "rm -rf #{dir} &2> /dev/null"
  end

  @db.close if @db
end


task :compile do
  compile_pdf(config)
end

task :view  => [:compile] do
  `open *.pdf`
end

task :init do
  terms = FileList["pages/*"].map do |file|
    content = File.open(file).read
    content.scan(/config\['(\w+)'\]/)
  end.flatten.uniq.sort

  terms << ""
  config_example = "---\n"
  config_example += terms.join(":\n")
  config_example += "# Note that the following items are arrays: amendments and project_milestones\n"
  config_example += "# Array items should be formatted as yaml arrays"

  File.write("config.yml.example", config_example)
end

task :sha do
  check_db_record
end

task :default => [:view]
