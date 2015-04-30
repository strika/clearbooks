# Clearbooks
Version 0.13.0-15-g9ff338a

[![Gem Version](https://badge.fury.io/rb/clearbooks.svg)](http://badge.fury.io/rb/clearbooks)
[![License](http://img.shields.io/badge/license-MIT-brightgreen.svg)](http://img.shields.io/badge/license-MIT-brightgreen.svg)

This is an unofficial Clear Books [1] API gem to handle any kind of interaction of their SOAP
Service [2] via a native Ruby interface. Clear Books is an online accounting software usable as a
Software as a Service (SaaS). Their official API works via SOAP and WSDL and currently is only officially supported
via PHP [3].

It allows the handling of invoices, expenses, financial accounts and mobile accounting as well as online HR and payroll.


[1] Clear Books PLC, https://www.clearbooks.co.uk

[2] https://www.clearbooks.co.uk/support/api

[3] https://www.clearbooks.co.uk/support/api/code-examples/php/

[4] https://github.com/clearbooks


## Features

- Application
  - Ruby VM (2.2 or better)
- Feature Providing Base Libraries
  - Savon
  - Thor
- Development
- Development Base Libraries
  - RVM
  - Rake
  - Thor
- Code Quality
  - Code review
  - Yard & related  (gem install yard --no-ri --no-rdoc ; gem install rdiscount --no-ri --no-rdoc)
  - MetricFu/RSpec/Cucumber


## Installing

By running gem comand

```sh
gem install clearbooks
```

or by adding to `Gemfile`

```ruby
gem 'clearbooks', github: 'greylon/clearbooks'
```

## Configuration

Go to Clearbooks http://clearbooks.co.uk and get your API key.

Save the API key in `~/.clearbooks/config.yml`.


```sh
$ echo "api_key: {your_api_key}" >> ~/.clearbooks/config.yml
```

Or provide the API key in `ENV['CLEARBOOKS_API_KEY']`

```sh
$ CLEARBOOKS_API_KEY=your_api_key clearbooks
```

Or use `Clearbooks.configure` block:

```ruby
require 'clearbooks'

Clearbooks.configure do |config|
    config.api_key = 'your_api_key'     # Unless you have key in ~/.clearbooks/config.yml
                                        # or ENV['CLEARBOOKS_API_KEY']
    config.log = true                   # If you need logging
    config.logger = Logger.new(STDOUT)  # Or any other logger of your choice
end
```

## Usage

### Ruby code

```ruby

Clearbooks.list_invoices        # returns Array of existing invoices
Clearbooks.list_entities        # returns Array of existing entities
Clearbooks.list_projects        # returns Array of existing projects
Clearbooks.list_account_codes   # returns Array of available account codes

Clearbooks.create_invoice Clearbooks::Invoice.new(date_created: Date.today,
      credit_terms: 30,
      entity_id: 1,
      type: :purchases,
      items: [
        Item.new(description: 'Item 1', unit_price: '9.99',
            quantity: 5, type: '1001001', vat: 0, vat_rate: '0.00:Out'),
        Item.new(description: 'Item 2', unit_price: '19.99',
            quantity: 7, type: '1001001', vat: 0, vat_rate: '0.00:Out')]
      ])
```
See the API reference below or visit the official Clearbooks site: https://www.clearbooks.co.uk/support/api/docs/soap/

### Command line
Type `clearbooks` or `clearbooks console` in the command line to launch [pry](https://github.com/pry/pry) in context of Clearbooks:
```sh
$ clearbooks
[1] pry(Clearbooks)> list_invoices
# ...
[1] pry(Clearbooks)> create_invoice Invoice.new(params)
# ...
[1] pry(Clearbooks)> exit
```

# Clearbooks API reference

Detailed API reference can be obtained from official Clearbooks site: https://www.clearbooks.co.uk/support/api/docs/soap/

## Managing invoices

### Clearbooks.list_invoices

Example:

```ruby
Clearbooks.list_invoices(
    id:             [1, 2, 3],          # Optional. Filter invoices by invoice id
    entity_id:      [1, 2, 3],          # Optional. Filter invoices by entity id
    ledger:         :sales,             # Optional. One of [:sales, :purchases]
    status:         :all,               # Optional. One of
                                        # [:draft, :voided, :bad_debt, :valid, :paid, :unpaid,
                                        #       :credited, :credit_note, :refund, :recurring]
    modified_since: '2015-01-01',       # Optional.
    offset: 0                          # Optional. It returns 100 invoices a time.
    ) # Clearbooks.list_invoices

 # Returns an Array of Invoice objects with attributes according to official API docs.
```
Reference: https://www.clearbooks.co.uk/support/api/docs/soap/listinvoices/

### Clearbooks.create_invoice

Example:

```ruby
Clearbooks.create_invoice Clearbooks::Invoice.new(
    date_created:   Date.today, # Reqiured. The tax point of the invoice.
    date_due:       Date.today, # The date the invoice is due.
    credit_terms:   30,         # The number of days after the tax point that the invoice is due.
                                # Either :date_due or :credit_terms is required.
    entity_id:      1,          # Required. The customer or supplier id.
    date_accrual:   Date.today, # Optional. The invoice accrual date.
    description:    'desc'      # Optional.
    type:           :purchases, # Optional. One of [:purchases, :sales, :cn-sales, :cn-purchases]
    bank_payment_id:200,        # Optional. The bank account code.
        # Can be extracted from the bank account URL:
        # 1. Go to Clearbooks site > Money > Bank accounts > All
        # 2. Click the bank account.
        # 3. In the address bar you will see the url like:
        #   https://secure.clearbooks.co.uk/company/accounting/banking/statement/7502001/
        # 4. The last number (7502001) is the bank account code.

    items: [
        Item.new(
            description: 'Item 1',  # Reqiured.
            unit_price: '9.99',     # Required.
            quantity: 5,            # Required.
            type: '1001001',        # Required. The item account code.
                                    # Use Clearbooks.list_account_codes
                                    # to get available account codes.
            vat: 0,                 # Reqiured.
            vat_rate: '0.00:Out'    # Required.
            )] # items
) # Clearbooks.create_invoice

# returns a Hash:

    {
        invoice_id: 1,
        invoice_prefix: 'INV',
        invoice_number: '1'
    }
```
Reference: https://www.clearbooks.co.uk/support/api/docs/soap/createinvoice/

## On what Hardware does it run?

This Software was originally developed and tested on 32-bit x86 / SMP based PCs running on
Gentoo GNU/Linux 3.13.x. Other direct Linux and Unix derivates should be viable too as
long as all dynamical linking dependencys are met.


## Documentation

A general developers API guide can be extracted from the Yardoc subdirectory which is able to
generate HTML as well as PDF docs. Please refer to the [Rake|Make]file for additional information
how to generate this documentation.

```sh
~# rake docs:generate
```

## Software Requirements

This package was developed and compiled under Gentoo GNU/Linux 3.x with the Ruby 2.x.x.
interpreter. It uses several libraries and apps as outlied in the INSTALLING section.

 - e.g. Debian/GNU Linux or Cygwin for MS Windows
 - Ruby
 - RVM or Rbenv
 - Bundler

## Configuration

Configuration is handled at run-time via $HOME/.clearbooks/config.yaml file.

## Build & Packaging

Package building such as RPM or DEB has not been integrated at this time.

To build the gem from this repository:


```sh
~# rake build
~# rake package
~# rake install
```

## Development

#### Software requirements

This package was developed and compiled under Gentoo GNU/Linux 3.x with the Ruby 2.x.x.
interpreter.

#### Setup

If you got this package as a packed tar.gz or tar.bz2 please unpack the contents in an appropriate
folder e.g. ~/clearbooks/ and follow the supplied INSTALL or README documentation. Please delete or
replace existing versions before unpacking/installing new ones.

Get a copy of current source from SCM

```sh
~# git clone ssh://github.com/greylon/clearbooks.git clearbooks
```

Install RVM (may not apply)

```sh
~# curl -sSL https://get.rvm.io | bash -s stable
```

Make sure to follow install instructions and also integrate it also into your shell. e.g. for ZSH,
add this line to your .zshrc.

```sh
[[ -s "$HOME/.rvm/scripts/rvm" ]] && source "$HOME/.rvm/scripts/rvm" ;

or

~# echo "source /usr/local/rvm/scripts/rvm" >> ~/.zshrc

```

or see `http://rvm.io` for more details.


Create proper RVM gemset

```sh
~# rvm --create use 2.2.2@clearbooks_project
```

Install Ruby VM 2.2.2 or better

```sh
~# rvm install 2.2.2
```

Install libraries via bundler

```sh
~# gem install bundler
~# bundle
```

#### Rake Tasks

For a full list of Rake tasks suported, use `rake -T`.

Here is a current listing of all tasks:


```
$rake_tasks$
```

#### Thor Tasks

For a full list of Thor tasks suported, use `thor -T`.

Here is a current listing of all tasks:


```
$thor_tasks$
```

## If something goes wrong

In case you enconter bugs which seem to be related to the package please check in
the MAINTAINERS.md file for the associated person in charge and contact him or her directly. If
there is no valid address then try to mail Bjoern Rennhak <bjoern AT greylon DOT com> to get
some basic assistance in finding the right person in charge of this section of the project.

## Contributing

1. Fork it ( https://github.com/greylon/clearbooks/fork )
2. Create your feature branch (`git checkout -b my_new_feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my_new_feature`)
5. Create a new Pull Request

## Authors

* [Oleg Kukareka](https://github.com/kukareka)
* [Bjoern Rennhak](https://github.com/rennhak)

## Copyright & License

Please refer to the COPYING.md and LICENSE.md file.
Unless otherwise stated in those files all remains protected and copyrighted by Bjoern Rennhak
(bjoern AT greylon DOT com).
