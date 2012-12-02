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

$views = Enlive::ViewGroup.new

# Let's use a "style guide" template from our designer
style_guide = "path/to/file.html" # relative to current directory

# We'll create a "view" based on the entire style guide template and
# call it `layout`:
$views.define(:layout, style_guide) do |title, main_content|
  with("head > title") { content(title) }
  with("body")         { content(main_content) }
end

# We'll create a "view" based on an example article that our designer
# added to the style guide template. We'll call it `article` and
# locate it in the template using the CSS selector `body article`:
$views.define(:article, style_guide, "body article") do |article|
  with("header h1")       { content(article[:title]) }
  with("section.content") { content(article[:body]) }

  with("header time") do
    attr(:datetime, article[:posted_at].iso8601)
    content(article[:posted_at].strftime("%b %d, %Y"))
  end
end

# Now we'll define a "view" for a list of articles. We'll base this
# off the `body > section#articles` element in our template. Let's
# call this "view" `article_list`.
$views.define(:article_list, style_guide, "body > section#articles") do |list_title, articles|
  with("> h1")      { content(list_title) }
  with(".articles") { empty } # Clear the children/content

  with(".articles") do
    articles.each do |article|
      # call our article view method (created above) for each article object
      # and add it as a child to the article list element
      append($views.article(article))
    end
  end
end

##################################################################

# Create a helper function to render the view
def articles_page(title, articles)
  $views.layout(title, $views.article_list(title, articles)).to_s
end

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

# Get the string of our assembled page, populated with our data, and
# designed by our designer, using their own optimized and ideal workflow
articles_page("My Latest Posts", blog_posts) # => "<html><head>..."
~~~

## Contributing

1. Fork it
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create new Pull Request

## License

See `LICENSE.txt`.
