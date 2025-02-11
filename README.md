# NAME

Auth::GoogleAuth - Google Authenticator TBOT Abstraction

# VERSION

version 1.06

[![test](https://github.com/gryphonshafer/Auth-GoogleAuth/workflows/test/badge.svg)](https://github.com/gryphonshafer/Auth-GoogleAuth/actions?query=workflow%3Atest)
[![codecov](https://codecov.io/gh/gryphonshafer/Auth-GoogleAuth/graph/badge.svg)](https://codecov.io/gh/gryphonshafer/Auth-GoogleAuth)

# SYNOPSIS

    use Auth::GoogleAuth;

    my $auth = Auth::GoogleAuth->new;

    $auth = Auth::GoogleAuth->new({
        secret => 'some secret string thing',
        issuer => 'Gryphon Shafer',
        key_id => 'gryphon@cpan.org',
    });

    $auth->secret();   # get/set
    $auth->secret32(); # get/set
    $auth->issuer();   # get/set
    $auth->key_id();   # get/set

    my $secret32 = $auth->generate_secret32;

    my $otpauth_0 = $auth->otpauth;
    my $otpauth_1 = $auth->otpauth(
        'bv5o3disbutz4tl3', # secret32
        'gryphon@cpan.org', # key_id
        'Gryphon Shafer',   # issuer
    );

    my $url_0 = $auth->qr_code;
    my $url_1 = $auth->qr_code(
        'bv5o3disbutz4tl3', # secret32
        'gryphon@cpan.org', # key_id
        'Gryphon Shafer',   # issuer
    );
    my $url_2 = $auth->qr_code(
        'bv5o3disbutz4tl3', 'gryphon@cpan.org', 'Gryphon Shafer', 1,
    );

    my $code_0 = $auth->code;
    my $code_1 = $auth->code( 'utz4tl3bv5o3disb', 1438643789, 30 );

    my $verification_0 = $auth->verify('879364');
    my $verification_1 = $auth->verify(
        '879364',           # code
        1,                  # range
        'utz4tl3bv5o3disb', # secret32
        1438643820,         # timestamp (defaults to now)
        30,                 # interval (default 30)
    );

    $auth->clear;

# DESCRIPTION

This module provides a simplified interface to supporting typical two-factor
authentication (i.e. "2FA") with
[Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator)
using the
[TOTP Algorithm](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)
as defined by [RFC 6238](http://tools.ietf.org/html/rfc6238).
Although Google Authenticator supports both TOTP and HOTP, at the moment,
this module only supports TOTP.

# METHODS

The following are the supported methods of this module:

## new

This is a simple instantiator to which you can pass optional default values.

    my $auth = Auth::GoogleAuth->new;

    $auth = Auth::GoogleAuth->new({
        secret => 'some secret string thing',
        issuer => 'Gryphon Shafer',
        key_id => 'gryphon@cpan.org',
    });

The object returned will support the following attribute get/set methods:

### secret

This can be any string. It'll be used as the internal secret key to create
the QR codes and authentication codes.

### secret32

This is a base-32 encoded copy of the secret string. If this is left undefined
and you run one of the methods that require it (like `qr_code` or `code`),
the method called will try to create the "secret32" by looking for a value in
"secret". If none exists, a random "secret32" will be generated.

### issuer

This is the label name of the "issuer" of the authentication.
See the
[key URI format wiki page](https://github.com/google/google-authenticator/wiki/Key-Uri-Format)
for more information.

### key\_id

This is the label name of the "key ID" of the authentication.
See the
[key URI format wiki page](https://github.com/google/google-authenticator/wiki/Key-Uri-Format)
for more information.

## generate\_secret32

This method will generate a reasonable random "secret32" value, store it in the
get/set method, and return it.

    my $secret32 = $auth->generate_secret32;

## otpauth

This method returns a generated otpauth key URI.

    my $otpauth_0 = $auth->otpauth;
    my $otpauth_1 = $auth->otpauth(
        'bv5o3disbutz4tl3', # secret32
        'gryphon@cpan.org', # key_id
        'Gryphon Shafer',   # issuer
    );

## qr\_code

This method will return a Quick Chart API URL that will return a QR code based
on the data either in the object or provided to this method.

    my $url_0 = $auth->qr_code;
    my $url_1 = $auth->qr_code(
        'bv5o3disbutz4tl3', # secret32
        'gryphon@cpan.org', # key_id
        'Gryphon Shafer',   # issuer
    );

You can optionally add a final true value, and if you do, the method will
return the generated otpauth key URI rather than the Quick Chart API URL.

    my $url_2 = $auth->qr_code(
        'bv5o3disbutz4tl3', 'gryphon@cpan.org', 'Gryphon Shafer', 1,
    );

## code

This method returns an authentication code, as if you were using
[Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator)
with the "secret32" value.

    my $code_0 = $auth->code;

You can optionally pass override values similar to `qr_code`:

    my $code_1 = $auth->code(
        'utz4tl3bv5o3disb', # secret32
        1438643789,         # timestamp (defaults to now)
        30,                 # interval (default 30)
    );

## verify

This method is used for verification of codes entered by a user. Pass in the
code (required) and optionally a range value and any override values.

    my $verification_0 = $auth->verify('879364');

The range value is useful because the algorithm checks codes that are time-
based. If clocks are not exactly in sync, it's possible that a "nearly valid"
code would be entered and should be accepted as valid but will be seen as
invalid. By passing in an integer as a range value, you can stipulate how
"fuzzy" the time should be. The default range is 0. A value of 1 will mean that
a code based on a time 1 iteration plus or minus should verify.

    my $verification_1 = $auth->verify(
        '879364',           # code
        1,                  # range
        'utz4tl3bv5o3disb', # secret32
        1438643820,         # timestamp (defaults to now)
        30,                 # interval (default 30)
    );

## clear

Given that the "secret" and "secret32" values may persist in this object, which
could be a bad idea in some contexts, this `clear` method lets your clear out
all attribute values.

    $auth->clear;

# TYPICAL USE-CASE

Typically, you're probably going to want to either randomly generate a secret or
secret32 (`generate_secret32`) for a user and store it, or use a specific value
or hash of some value as the secret. In either case, once you have a secret and
its stored, generate a QR code (`qr_code`) for the user. You can alternatively
provide the "secret32" to the user for them to manually enter it. That's it
for setup.

To authenticate, present the user with a way to provide you a code (which will
be a series of 6-digits). Verify that code (`verify`) with either no range
or some small range like 1.

# DEPENDENCIES

[Digest::HMAC\_SHA1](https://metacpan.org/pod/Digest%3A%3AHMAC_SHA1), [Math::Random::MT](https://metacpan.org/pod/Math%3A%3ARandom%3A%3AMT), [URI::Escape](https://metacpan.org/pod/URI%3A%3AEscape), [Convert::Base32](https://metacpan.org/pod/Convert%3A%3ABase32),
[Class::Accessor](https://metacpan.org/pod/Class%3A%3AAccessor), [Carp](https://metacpan.org/pod/Carp).

# SEE ALSO

You can look for additional information about this module at:

- [GitHub](https://github.com/gryphonshafer/Auth-GoogleAuth)
- [MetaCPAN](https://metacpan.org/pod/Auth::GoogleAuth)
- [GitHub Actions](https://github.com/gryphonshafer/Auth-GoogleAuth/actions)
- [Codecov](https://codecov.io/gh/gryphonshafer/Auth-GoogleAuth)
- [CPANTS](http://cpants.cpanauthors.org/dist/Auth-GoogleAuth)
- [CPAN Testers](http://www.cpantesters.org/distro/G/Auth-GoogleAuth.html)

You can look for additional information about things related to this module at:

- [TOTP Algorithm](https://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm)
- [RFC 6238](http://tools.ietf.org/html/rfc6238)
- [Google Authenticator](https://en.wikipedia.org/wiki/Google_Authenticator)
- [Google Authenticator GitHub](https://github.com/google/google-authenticator)
- [Quick Chart QR Codes](https://quickchart.io/documentation/qr-codes)

# AUTHOR

Gryphon Shafer <gryphon@cpan.org>

# COPYRIGHT AND LICENSE

This software is Copyright (c) 2015-2050 by Gryphon Shafer.

This is free software, licensed under:

    The Artistic License 2.0 (GPL Compatible)
