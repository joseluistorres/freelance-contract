#!/usr/bin/env ruby
require 'pathname'
require 'erb'
require 'yaml'

if File.exists?('config.yml')
  config = YAML.load_file(File.expand_path 'config.yml')
else
  puts "Run `rake init` && `mv config.yml.example config.yml`"
  puts "Then fill in values as needed in config.yml"
  exit(1)
end


def compile_pdf(config)
  config = config
  contract_output_name = "Contract-#{Time.now.strftime('%Y-%m-%d-%H:%M:%S')}"

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

ensure
  post_hook
end

def process_erb(basename=basename, file_markdown=file_markdown, config)

  rendered_template = ERB.new(File.read("pages/#{basename}")).result(config.send(:binding))
  File.write("markdown/#{file_markdown}", rendered_template)

end

def pre_hook(contract_output_name)
  contract_output_name = contract_output_name
  system "rm *.pdf &2> /dev/null"
  system "rm *.sha1 &2> /dev/null"
  system "mkdir html" unless Dir.exists?("html")
  system "mkdir markdown" unless Dir.exists?("markdown")
end

def post_hook
  system "rm -rf markdown"
  system "rm -rf html"
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

task :default => [:view]
