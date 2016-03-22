# NAME

Dancer::Plugin::ValidateTiny - Validate::Tiny Dancer plugin.

# VERSION

Version 0.06

# SYNOPSIS

Easy and cool validating data with [Validate::Tiny](https://metacpan.org/pod/Validate::Tiny) module:

    use Dancer::Plugin::ValidateTiny;

    post '/' => sub {
        my $params = params;
        my $data_valid = 0;

        # Validating params with rule file
        my $data = validator($params, 'form.pl');

        if($data->{valid}) { ... }
    };

Rule file is pretty too:

    {
        # Fields for validating
        fields => [ qw/login email password password2/ ],
        filters => [
            qr/.+/ => filter(qw/trim strip/),
            email => filter('lc'),
        ],
        checks => [
            [ qw/login email password password2/ ] => is_required("Field required!"),

            login => is_long_between( 2, 25, 'Your login should have between 2 and 25 characters.' ),
            email => sub {
                check_email($_[0], "Please enter a valid email address.");
                },
            password => is_long_between( 4, 40, 'Your password should have between 4 and 40 characters.' ),
            password2 => is_equal("password", "Passwords don't match"),
        ],
    }

Note, that `@_` in anonymous sub in `checks` section contains value to be checked and a reference to the filtered input hash. Check [Validate::Tiny](https://metacpan.org/pod/Validate::Tiny) documentation for this.

# DESCRIPTION

Simple Dancer plugin for use [Validate::Tiny](https://metacpan.org/pod/Validate::Tiny) module.

It provides simple use for [Validate::Tiny](https://metacpan.org/pod/Validate::Tiny) way of validating user input with Dancer applications with some additions and modifications, such as separate files for rules and additional functions for data validation.

# METHODS

## validator

This is the main method, that receiving `params` from `POST` or `GET` (or whatever), and filename, which contains rules for validation:

    my $params = params;
    my $data = validator($params, 'form.pl');

After this, in `$data` you'll have a structure like:

    {
      'valid' => 0,
      'result' => {
                  'err_login' => 'Your login should have between 4 and 25 characters.',
                  'err_email' => 'Please enter a valid email address.',
                  'err_password' => 'Field required!'
                  'login' => 'foo',
                  'email' => 'test input',
                  'password' => ''
                }
    };

Where `valid` field is an indicator, that you can use like `if($data->{valid}) { ... }`.

And `result` field, that contains **already filtered** params and error messages for them with special prefixes. Note, that you can set up ["error\_prefix"](#error_prefix) in config file.

# RULE FILES

In your Dancer application directory you need to create sub-directory for rule files and place here rules, that you will use for validation. In this files you need to create a simple structure like this one:

    {
        fields => [qw/city zip_code/],
        checks => [
            [qw/city zip_code/] => is_required("Field required!"),

            city => is_long_at_most( 40, 'City name is too long' ),
            zip_code => is_long_at_least( 5, 'Bad zip code' ),
        ],
    }

For other rules, you can refer to the documentation of [Validate::Tiny](https://metacpan.org/pod/Validate::Tiny) module.

After creating rule file, you just need to specify it's name in ["validator"](#validator) method. Simple, yeah? :)

# ADDITIONAL RULES

There is some additional subroutines, that you can use in rule files:

## check\_email

    {
        fields => "email",
        filters => [
            email => filter('trim', 'strip', 'lc')
        ],
        checks => [
            email => sub { check_email($_[0], "Please enter a valid e-mail address.") },
        ],
    }

This subroutine checking e-mail address conforms to the RFC822 specification with [Email::Valid](https://metacpan.org/pod/Email::Valid).

Note, that `checks` processing data, that **already filtered** by `filters`.

# CONFIG

In your config file you can use these settings:

    plugins:
      ValidateTiny:
        rules_dir: validation
        error_prefix: err_
        is_full: 0

Where:

## rules\_dir

Directory, where you will store your rule files. Plugin looking it in your Dancer application root.

## error\_prefix

Prefix, that used to separate error fields from normal values in `result` hash. It is very convenient when you use the template engine, such as [Template::Toolkit](https://metacpan.org/pod/Template::Toolkit) or [HTML::Template](https://metacpan.org/pod/HTML::Template). You simply pass the data to the template engine, and it handles the logic of output errors and/or warnings for user.

## is\_full

If this option is set to `1`, call of `validator` returning an object, that you can use as standart [Validate::Tiny](https://metacpan.org/pod/Validate::Tiny) object.

# SEE ALSO

[Validate::Tiny](https://metacpan.org/pod/Validate::Tiny) [Dancer::Plugin::FormValidator](https://metacpan.org/pod/Dancer::Plugin::FormValidator) [Dancer::Plugin::DataFu](https://metacpan.org/pod/Dancer::Plugin::DataFu)

# AUTHOR

Alexey Kolganov, `<kalgan@cpan.org>`

# COPYRIGHT AND LICENSE

Copyright (C) 2011 by Alexey Kolganov

This library is free software; you can redistribute it and/or modify it under the same terms as Perl itself, either Perl version 5.10.1 or, at your option, any later version of Perl 5 you may have available.
