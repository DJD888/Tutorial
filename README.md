require 'mechanize'
require 'set'
require 'nokogiri'
require 'open-uri'
require 'open_uri_redirections'

class Scraper
  attr_accessor :url

  def initialize(url)
    @url = url
    @filtered_links = Set.new
    @links = Set.new
    @data = []
  end

  def query_url(depth)
    case depth
    when 1
    begin
      agent = Mechanize.new.get @url
      agent.links.each do |link|
        if link.class == Mechanize::Page::Link && link.uri.to_s.include?('http')
	  @links << link.uri.to_s
	end
    rescue OpenURI::HTTPError
    end
    when 2
    begin
      agent = Mechanize.new.get @url
      agent.links.each do |link|
        if link.class == Mechanize::Page::Link && link.uri.to_s.include?('http')
	  @links << link.uri.to_s
	end
      end
      @links.each do |link|
        agent = Mechanize.new.get link
	if link.class == Mechanize::Page::Link && link.uri.to_s.include?('http')
	  @links << link.uri.to_s
	end
      end
      rescue OpenURI::HTTPError
    end
  end

  def display_links
    p @links
  end

  def display_filtered_links
    p @filtered_links
  end

  def filter_links(filter)
    @links.each do |link|
      if link.include?(filter)
        @filtered_links << link
      end
    end
  end

  def scrape(selector)
  begin
    @filtered_links.each do |link|
      agent = Nokogiri::HTML(open(link, :allow_redirections => :safe))
      p selector
      agent.xpath(selector).each do |node|
        @data << node.content
      end
    end
  rescue OpenURI::HTTPError
  end
  end

  def write_scraped_data(filepath)
    File.open('output.txt', 'w') do |file|
      @data.each do |data|
        file.puts data + "\n"
      end
    end
  end

end# Tutorial
Broweber
