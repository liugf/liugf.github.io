---
layout: post
title: "Bending CSVs to your will with Ruby"
date: 2010-09-05 19:46
tags: Ruby CSV
description: "Ruby script to convert CSV file into various text formats for use in an Adobe InDesign project"
keywords: Ruby, CSV, InDesign, Text
image: "http://images.darrenknewton.com/rubycsv.png"
categories: jekyll update
---

{% include post_pic.html url='http://images.darrennewton.com/heroku.jpg' title='hello' %}

I'm currently working on a large print project. It's a membership directory for a non-profit organization and I'm laying it out with <a href="http://www.adobe.com">Adobe InDesign</a>. The client handed me a CSV dump of their member database for the directory. I've mentioned more times than I can count how much I dislike repetition (cutting, pasting, rinse, repeat) so I cooked up some <a href="http://www.ruby-lang.org/">Ruby</a> scripts to parse the CSV into various XML and plaintext formats. READMORE

## Hash it.

<p>To make life easier I converted the entire CSV file into an <a href="http://www.ruby-doc.org/core/classes/Array.html">array</a> of <a href="http://www.ruby-doc.org/core/classes/Hash.html">hashes</a> like so:</p>

        :::ruby
        require 'csv'

        def csv_to_array(file_location)
         csv = CSV::parse(File.open(file_location, 'r') {|f| f.read })
         fields = csv.shift
         csv.collect { |record| Hash[*fields.zip(record).flatten ] } 
        end

        test = csv_to_array('test.csv')


<p>This gets the CSV into a nice array loaded with hashes which names each entry with its corresponding field:</p>

        :::ruby
        {"name"=>"Larry", "company"=>"Google"}
        {"name"=>"Curly", "company"=>"Apple"}
        {"name"=>"Moe", "company"=>"Microsoft"}


## Re-shape it

<p>Getting the CSV into an array of hashes is pretty convenient. I can now rip through the hashes and do whatever formatting I need. For instance, I needed to create a plaintext file that converted each row in the CSV into a <code>Name, Company</code> format followed by a line break, which could easily be pasted into an InDesign text box:</p>

        :::ruby
        def csv_to_text(file_location)
          out = ""
          rows = csv_to_array(file_location) # get our CSV into an array of hashes
          rows.each do |row|

          # Use a HEREDOC to get the formatting we want

        tmp = &lt;&lt;HERE      
        #{row["name"]}, #{row["company"]}
        HERE

          out = out + tmp
          end

          save_text(file_location, out) # save a text file

        end


<p>This is all pretty simple, but it saves on a lot of cutting and pasting text from fields in Excel or Numbers, the kind of thing I really dislike. You can also do a lot of text transformation during the same process. Many of the addresses in the CSV needed to be made consistent. I wanted common words like Street, Boulevard, Avenue to get transformed into common abbreviations like St., Blvd., and Ave. We need a function:</p>

        :::ruby
        def process_address(data)
          if data.nil?
            return data
          end

          data.squeeze!(" ") # remove any double spaces
          data.strip!        # remove trailing spaces
          data.gsub!(/Res: /, '')
          data.gsub!(/Court/,'Ct.')
          data.gsub!(/Suite/, 'Ste.')
          data.gsub!(/Drive/, 'Dr.')
          data.gsub!(/Road/, 'Rd.')
          data.gsub!(/Street/, 'St.')
          data.gsub!(/Lane/, 'Ln.')
          data.gsub!(/Avenue/, 'Ave.')

          data  
        end


<p>Now we can pass our rows through <code>process_address</code> when writing to the text file to get consistent abbreviations. (I could probably optimize that function by dropping the abbreviations into a hash and iterating through them.)</p>

<p>These are pretty simple little scripts, but they help to quickly automate some very tedious tasks. They're also an example of how scripting languages can help you tackle data entry issues when working with your design tools. For this same project I also needed to generate a large XML file that InDesign could parse and use to build the member directory on its own. I'll cover that chunk of code in my next post.</p>