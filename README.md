# Enlive

HTML templating library that lets you create composable pure functions
for your views based on applying modifications to HTML templates using selectors.

Loosely based on the templating portion of Christophe Grand's
fantastic (if under-documented and under-appreciated) [Enlive](https://github.com/cgrand/enlive) for [Clojure](http://clojure.org).

## Installation

Add this line to your application's Gemfile:

    gem 'enlive'

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install enlive

## Usage

~~~ ruby
require 'enlive'

blog_posts = [
  {
    title: "My First Post",
    body: "<a href=\"http://github.com/bjeanes\">click here</a>",
    posted_at: Time.now - 1.hour
  },
  {
    title: "My Second Post",
    body: "<a href=\"http://github.com/bjeanes\">click here again</a>",
    posted_at: Time.now
  }
]

$views = Enlive::ViewGroup.new
style_guide = "path/to/file.html" # relative to current directory

$views.deftemplate(:layout, style_guide) do |title, content|
  with("head > title") { |title_el| title_el.content = title }
  with("body")         { |body| body.children = content }
end

$views.defsnippet(:article, style_guide, "body article") do |article|
  with("header h1")       { |h1| h1.content = article[:title] }
  with("section.content") { |sec| sec.content = article[:body] }
  with("header time") do |time|
    time[:datetime] = article[:posted_at].iso8601
    time.content    = article[:posted_at].strftime("%b %d, %Y")
  end
end

$views.defsnippet(:article_list, style_guide, "body > section#articles") do |list_title, articles|
  with("> h1")      { |h1| h1.content = list_title }
  with(".articles") { |list| list.delete }

  with(".articles") do |list|
    articles.each do |article|
      # create an article snippet from each article object
      list << $views.article(article)
    end
  end
end

# Create a helper function to render the view
def articles_page(title, articles)
  $views.layout(title, $views.article_list(title, articles)).to_s
end
~~~

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

See `LICENSE.txt`.
