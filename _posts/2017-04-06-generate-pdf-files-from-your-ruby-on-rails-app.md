---
layout: post
title: "Generate PDF files from your Rails app"
date: "2017-04-06"
---

Working recently on a Rails project, I needed to generate PDF files from the app. After searching on Github (specifically in [Awesome Ruby](https://github.com/markets/awesome-ruby#pdf)) for open source solutions that fit my needs , I ended up choosing [wicked_pdf](https://github.com/mileszs/wicked_pdf). Let's see how to make the magic happen.

## Why wicked_pdf ?

I chosen Wicked PDF for 3 specific reasons,

- It's very easy to use
- It helps you generate your PDF files from HTML & CSS (& JS)
- It's easy to integrate into a Rails project (It's actually a [Rails plugin](http://guides.rubyonrails.org/plugins.html))

Wicked_pdf uses the awesome [wkhtmltopdf](https://github.com/wkhtmltopdf/wkhtmltopdf) project, to create PDF files. The wkhtmltopdf itself, is a command line tool to render HTML into PDF using the [QT webkit](https://wiki.qt.io/Qt_WebKit) rendering engine.

## Install

### wicked_pdf

Add this to your Gemfile, then run  `bundle install` :
```
gem 'wicked_pdf'
```

### wkhtmltopdf

Because wicked_pdf is a wrapper of wkhtmltopdf, you'll need to install it too. You can install it through the `wkhtmltopdf-binary` by adding this to your Gemfile and running `bundle install`:
```
gem 'wkhtmltopdf-binary'
```

For one reason or another you may want to install it manually instead. In that case, you will have to tell wicked_pdf where the executable is located. Run `rails g wicked_pdf` to generate an initializer for wicked_pdf and then add:
```
WickedPdf.config = {
  exe_path: '/usr/local/bin/wkhtmltopdf' # The value of exe_path being the path to the wkhtmltopdf binary
}
```

## Render a view as pdf

To render a  `show.html.erb` view as pdf, we'll simply do:
```
class MyController < ApplicationController
  def show
    format.pdf do
      render pdf: 'file_name'   # Excluding ".pdf" extension.
    end
  end
end
```

Because the wkhtmltopdf binary works outside of the rails app, it won't be able to load our app assets if the view (or the view's layout) includes any. Let's fix that by creating a special layout for our PDF files. Assets can be included into the layout either using wicked_pdf helpers or CDN references.

### Create layout using wicked_pdf helpers

wicked_pdf makes available 3 helpers: **wicked_pdf_stylesheet_link_tag**, **wicked_pdf_javascript_tag**, **wicked_pdf_image_tag** that we can use to include our assets as one would do with the Rails asset pipeline methods.

```ruby
//pdf_layout.pdf.erb

<!doctype html>
<html>
  <head>
    <meta charset='utf-8' />
    <%= wicked_pdf_stylesheet_link_tag "app_css_file" -%>
    <%= wicked_pdf_javascript_include_tag "app_js_file" %>
  head>
  <body>
    <div id="header">
      <%= wicked_pdf_image_tag 'pdf_header_image.jpg' %>
    div>
    <div id="content">
      <%= yield %>
    <div>
  <body>
<html>
```

### Create layout using CDN references

In the case that the assets files are libraries or frameworks available on CDNs, we can simply use the Rails asset tag helpers (**stylesheet_link_tag**, **javascript_include_tag**, **image_tag)** to include them.
```ruby
//pdf_layout.pdf.erb

<!doctype html>
<html>
  <head>
    <meta charset='utf-8' />
    <%= stylesheet_link_tag "https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css" -%>
    <%= javascript_include_tag "https://code.jquery.com/jquery-3.2.1.min.js" %>
  <head>
  <body>
    <div id="header">
      <%= image_tag "https://placehold.it/350x150" %>
    <div>
    <div id="content">
      <%= yield %>
    <div>
  <body>
html>
```

### Rendering with the layout

class MyController < ApplicationController
  def show
    format.pdf do
      render pdf: 'file_name',
             layout: 'pdf_layout' # for a file named pdf_layout.pdf.erb
    end
  end
end

## Advanced usage

If you would like to create a PDF file and make another use of it instead of directly rendering it, or if you want to control page breaks and page numbering, read the  [project's README advanced usage section](https://github.com/mileszs/wicked_pdf#advanced-usage-with-all-available-options).

## Other issues

### CSS Flexbox not supported

I found out that, at the time of this writing, the version of Qt Webkit used in the wkthmltopdf binary doesn't support the CSS Flexbox specification. To take advantage of Flexbox in your PDF files, I recommend using a polyfill as the [awesome flexibility library](https://github.com/jonathantneal/flexibility)  in your views.

### PDF files generation time

Depending on the contents involved, the server can take a long time to generate a PDF file. Thus, it's better to put the generation logic into a job instead of the request/response cycle. See the [Ruby on Rails guide](http://guides.rubyonrails.org/active_job_basics.html) to learn more about jobs.

Thanks for reading !
