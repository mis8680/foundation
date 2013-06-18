#!/usr/bin/env rake
require "bundler/gem_tasks"
require "./lib/foundation/version"
require "./lib/foundation/utils/sass_variable_extractor"
require "./lib/foundation/utils/js_variable_extractor"

def update_sass_variables_from_components
  output_file = File.new("./scss/foundation/_variables.scss", "w")
  sass_files = Dir.glob("./scss/foundation/components/**/_*.scss")
  sass_files.each do |pth|
    sve = SassVariableExtractor.new(pth)
    if vars = sve.extract_sass_variables
      output_file.puts "// #{File.basename(pth)}"
      vars.each do |v|
        line = "// $#{v[:name]}: #{v[:value]}; #{}"
        line += " // #{v[:comment]}" if v[:comment]
        output_file.puts line
      end
      output_file.puts "\n"
    end
  end
  output_file.close
  # puts "Output written to: #{output_file.path}"
end

def update_js_versions
  major_num = Foundation::VERSION.split(".").first
  branch = `git branch`.lines.select {|l| l.start_with?("*")}.first.chop.split("* ").last
  tags = `git tag`.split("\n").map(&:chomp).select {|t| t.start_with?("v#{major_num}")}

  if tags.include?("v#{Foundation::VERSION}")
    puts "Please update Foundation::VERSION (currently #{Foundation::VERSION})"
    return
  end

  # If no major tags available, then update everything
  if tags.select {|t| t.start_with?("v#{major_num}")}.empty?
    puts "This is the first major release, please update everything"
    return
  end

  # Look for commit ID of the most recent tag
  # puts "Released version: #{tags.max}"
  # puts "Current version: v#{Foundation::VERSION}"

  tag = tags.max
  files = `git diff --name-only #{tag} HEAD`.split("\n").map(&:chomp)


  # JS files that changed
  js_files = files.select {|pth| pth.start_with?("js/") && File.extname(pth) === ".js"}
  sass_files = files.select {|pth| pth.start_with?("scss/") && File.extname(pth) === ".scss"}

  # Increment JS File Versions
  js_files.each do |pth|
    extractor = JsVariableExtractor.new(pth)
    extractor.set_version!(Foundation::VERSION)
  end
end

desc "Update versions"
task :update do
  update_sass_variables_from_components
  update_js_versions
  puts "Updated to v#{Foundation::VERSION}"
end