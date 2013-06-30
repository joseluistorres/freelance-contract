#!/usr/bin/env ruby
require 'pathname'
require 'erb'
require 'yaml'

config = YAML.load_file(File.expand_path 'config.yml')

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
      files << "'html/#{file_html}' " unless basename == '00-Title Page.md.erb'
    # end
  end

  html_doc =  "htmldoc --book --titlefile 'html/00-Title Page.html'"
  html_doc += " -f #{ contract_output_name }.pdf #{files}"

  system html_doc

  system "shasum #{contract_output_name}.pdf > #{contract_output_name}.sha"

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
  system "rm *.sha &2> /dev/null"
  system "mkdir html" unless Dir.exists?("html")
  system "mkdir markdown" unless Dir.exists?("markdown")
end

def post_hook
  system "rm -rf markdown"
  system "rm -rf html"
end

task :default => [:view]

task :compile do
  compile_pdf(config)
end

task :view  => [:compile] do
  `open *.pdf`
end
