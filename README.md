# Rails Excel

It adds support of .rxls templates for your rails views

It comes with two builtin strategies based on the very good gems [writeexcel](https://github.com/cxn03651/writeexcel) and [spreadsheet](http://spreadsheet.rubyforge.org/index.html)

# Requirements

* rails ~> '2.3.0'
* ruby ~> '1.8.6'


# Usage

In your config/environment.rb

    config.gem 'rails-excel'


Create an initializer : config/initializers/excel.rb

    Rails::Excel.configure do |config|
      config.strategy = :spreadsheet # by default or :write_excel
    end


If you wan tot implement your own strategy, here are the requirements :

* Must respond to compile
* First argument of compile takes a StringIo instance where to write the response
* compile takes a block that yields a workbook instance


Example extracted from code source:

    # lib/my_strategy.rb

    class MyStrategy
      def compile(io, &block)
        workbook = ::Spreadsheet::Workbook.new
        yield(workbook)
        workbook.write(io)
      end
    end

Then in your config/initializers/excel.rb


    require 'my_strategy'
    Rails::Excel.configure do |config|
      config.add_strategy :my_strategy, MyStrategy.new
      # Redefining default strategy
      config.strategy = :my_strategy # by default it was :spreadsheet
    end

You can use any object as long as it responds to described `compile` method


The strategy defined in the initializer will be used in all of your controllers

To use another strategy you can set it by controller :

    class UsersController < ApplicationController
      self.excel_strategy = :write_excel
    end


Or you can set strategy per action by redefining instance method excel_strategy

    class UsersController < ApplicationController

      self.excel_strategy = :write_excel

      def index
        # ...
      end

      def show
        # ...
      end

      protected
      def excel_strategy
        case action_name
        when 'index'
          :spreadsheet
        else
          self.class.excel_strategy
        end
      end

    end
