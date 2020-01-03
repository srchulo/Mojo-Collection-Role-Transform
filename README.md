# NAME

Mojo::Collection::Role::Transform - Transformations for Mojo::Collection

# STATUS

<div>
    <a href="https://travis-ci.org/srchulo/Mojo-Collection-Role-Transform"><img src="https://travis-ci.org/srchulo/Mojo-Collection-Role-Transform.svg?branch=master"></a> <a href='https://coveralls.io/github/srchulo/Mojo-Collection-Role-Transform?branch=master'><img src='https://coveralls.io/repos/github/srchulo/Mojo-Collection-Role-Transform/badge.svg?branch=master' alt='Coverage Status' /></a>
</div>

# SYNOPSIS

    my $c = Mojo::Collection->new(
      {
        name           => 'Bob',
        age            => 23,
        favorite_color => 'blue',
      },
      {
        name           => 'Alice',
        age            => 24,
        favorite_color => 'blue',
      },
      {
        name           => 'Eve',
        age            => 27,
        favorite_color => 'green',
      },
    )->with_roles('+Transform');

    # hash key is name, value is the original hash
    my $name_to_person = $c->hashify(sub { $_->{name} });

    # 23
    say $name_to_person->{Bob}{age};

    # 27
    say $name_to_person->{Eve}{age};

    # set your own value
    my $name_to_favorite_color = $c->hashify(sub { $_->{name} }, sub { $_->{favorite_color} });

    # blue
    say $name_to_favorite_color->{Bob};

    # green
    say $name_to_favorite_color->{Eve};


    # collect values with the same key in a Mojo::Collection
    my $favorite_color_to_collection = $c->hashify_collect(sub { $_->{favorite_color} });

    # $favorite_color_to_collection->{blue} contains Mojo::Collection of Bob and Alice hashes
    # $favorite_color_to_collection->{green} contains Mojo::Collection of Eve hash

    # says Bob, then Alice
    for my $person ($favorite_color_to_collection->{blue}->each) {
      say $person->{name};
    }


    # Create a Mojo::Collection of Mojo::Collections, where all values in each inner
    # Mojo::Collection share the same key
    my $collections_by_favorite_color = $c->collect_by(sub { $_->{favorite_color} });
    for my $favorite_color_collection ($collections_by_favorite_color->each) {
      say $favorite_color_collection->[0]->{favorite_color};

      for my $person ($favorite_color_collection->each) {
        say "\t$person->{name}";
      }
    }

    # output is
    # blue
    #     Bob
    #     Alice
    # green
    #     Eve

# DESCRIPTION

[Mojo::Collection::Role::Transform](https://metacpan.org/pod/Mojo::Collection::Role::Transform) provides methods that allow you to transform your [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection) in meaningful and flexible ways.

# METHODS

## hashify

- hashify($get\_keys\_sub, \[$get\_value\_sub\])

    my $c = Mojo::Collection->new(
      {
        name           => 'Bob',
        age            => 23,
        favorite_color => 'blue',
      },
      {
        name           => 'Alice',
        age            => 24,
        favorite_color => 'blue',
      },
      {
        name           => 'Eve',
        age            => 27,
        favorite_color => 'green',
      },
    )->with_roles('+Transform');

    # hash key is name, value is the original hash
    my $name_to_person = $c->hashify(sub { $_->{name} });

    # 23
    say $name_to_person->{Bob}{age};

    # 27
    say $name_to_person->{Eve}{age};


    # set your own value
    my $name_to_favorite_color = $c->hashify(sub { $_->{name} }, sub { $_->{favorite_color} });

    # blue
    say $name_to_favorite_color->{Bob};

    # green
    say $name_to_favorite_color->{Eve};


    # return multiple keys as a list to create a multiple nested hashes based on the returned keys and their order
    my $name_to_age_favorite_color = $c->hashify(sub { @$_{qw(name age)} }, sub { $_->{favorite_color} });

    # blue
    say $name_to_favorite_color->{Bob}{23};

    # green
    say $name_to_favorite_color->{Eve}{27};

["hashify"](#hashify) allows you to transform a [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection) into a single key or multi-key hash based on its elements. A unique
key or list of keys may only have one value, so for any duplicate keys, the final element seen with that key will be the one that sets the
value.

### get\_keys

["get\_keys"](#get_keys) is required and must return a single key or a list of keys. The return value of ["get\_keys"](#get_keys) will be the keys used to
ultimately access the value that is returned by ["get\_value"](#get_value) for the same element.

    # return single key
    my $hash = $c->hashify(sub { $_->{name} });
    my $value = $hash->{$name};

    # return multiple keys
    my $hash = $c->hashify(sub { $_->{name}, $_->{age} });
    my $value = $hash->{$name}{$age};

The current element is available via `$_`, or as the first argument to ["get\_keys"](#get_keys).

### get\_value

["get\_value"](#get_value) must return a **single** value for the current element in the [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection).

    my $name_to_age = $c->hashify(sub { $_->{name} }, sub { $_->{age} });
    my $age = $name_to_age->{Bob};

The default is to return the current element:

    # default get_value
    sub { $_ }

    # not passing in get_value uses the above subroutine to return the current collection element, in this case a hash
    my $name_to_person = $c->hashify(sub { $_->{name} });
    my $age = $name_to_person->{Bob}{age};

The current element is available via `$_`, or as the first argument to ["get\_value"](#get_value).

## hashify\_collect

- hashify\_collect($get\_keys\_sub, \[$get\_values\_sub\])

    my $c = Mojo::Collection->new(
      {
        name           => 'Bob',
        age            => 23,
        favorite_color => 'blue',
      },
      {
        name           => 'Alice',
        age            => 24,
        favorite_color => 'blue',
      },
      {
        name           => 'Eve',
        age            => 27,
        favorite_color => 'green',
      },
    )->with_roles('+Transform');

    # collect values with the same key in a Mojo::Collection
    my $favorite_color_to_collection = $c->hashify_collect(sub { $_->{favorite_color} });

    # $favorite_color_to_collection->{blue} contains Mojo::Collection of Bob and Alice hashes
    # $favorite_color_to_collection->{green} contains Mojo::Collection of Eve hash

    # says Bob, then Alice
    for my $person ($favorite_color_to_collection->{blue}->each) {
      say $person->{name};
    }

    # provide your own get_values sub
    my $favorite_color_to_names = $c->hashify_collect(sub { $_->{favorite_color} }, sub { $_->{name} });

    # $favorite_color_to_names->{blue} contains Mojo::Collection of 'Bob' and 'Alice'
    # $favorite_color_to_names->{green} contains Mojo::Collection of 'Eve'

    # return multiple values
    my $favorite_color_to_names_and_ages = $c->hashify_collect(sub { $_->{favorite_color} }, sub { $_->{name}, $_->{age} });

    # $favorite_color_to_names_and_ages->{blue} contains Mojo::Collection of 'Bob', 23, 'Alice', 24
    # $favorite_color_to_names_and_ages->{green} contains Mojo::Collection of 'Eve', 27

["hashify\_collect"](#hashify_collect) allows you to transform a [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection) into a single key or multi-key hash where the final value is a [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection) with
all elements that match that key. ["get\_values"](#get_values) allows you to control which values are collected in the [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection).

### get\_keys

["get\_keys"](#get_keys1) is required and must return a single key or a list of keys. The return value of ["get\_keys"](#get_keys1) will be the keys used to
ultimately access the [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection) of values that are returned by ["get\_values"](#get_values) for each element.

    # return single key
    my $hash = $c->hashify(sub { $_->{name} });
    my $collection = $hash->{$name};

    # return multiple keys
    my $hash = $c->hashify(sub { $_->{name}, $_->{age} });
    my $collection = $hash->{$name}{$age};

The current element is available via `$_`, or as the first argument to ["get\_keys"](#get_keys1).

### get\_values

["get\_values"](#get_values) may return one or more values as a list for the current element in the [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection).

    # return single value
    my $name_to_ages = $c->hashify(sub { $_->{name} }, sub { $_->{age} });
    my $ages_collection = $name_to_ages->{Bob};

    # return multiple values
    my $name_to_ages_and_favorite_colors = $c->hashify(sub { $_->{name} }, sub { $_->{age}, $_->{favorite_color} });
    my $ages_and_favorite_colors_collection = $name_to_ages_and_favorite_colors->{Bob};

The default is to return the current element:

    # default get_value
    sub { $_ }

    # not passing in get_value uses the above subroutine to return the current collection element, in this case a hash
    my $name_to_persons = $c->hashify(sub { $_->{name} });
    my $person_collection = $name_to_persons->{Bob};

The current element is available via `$_`, or as the first argument to ["get\_values"](#get_values).

### OPTIONS

#### flatten

    # trivial example where returned values are wrapped in arrayrefs to demonstrate flatten
    my $name_to_ages = $c->hashify({flatten => 1}, sub { $_->{name} }, sub { [$_->{age}] });

The ["flatten"](#flatten) option flattens each resulting [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection), meaning that
it flattens nested collections/arrays recursively and creates a new collection with all elements. See ["flatten" in Mojo::Collection](https://metacpan.org/pod/Mojo::Collection#flatten)
for more details.

Internally, ["flatten"](#flatten) is implemented differently for performance, but the end result is the same.

## collect\_by

- collect\_by($get\_keys\_sub, \[$get\_values\_sub\])

    my $c = Mojo::Collection->new(
      {
        name           => 'Bob',
        age            => 23,
        favorite_color => 'blue',
      },
      {
        name           => 'Alice',
        age            => 24,
        favorite_color => 'blue',
      },
      {
        name           => 'Eve',
        age            => 27,
        favorite_color => 'green',
      },
    )->with_roles('+Transform');

    # Create a Mojo::Collection of Mojo::Collections, where all values in each inner
    # Mojo::Collection share the same key
    my $collections_by_favorite_color = $c->collect_by(sub { $_->{favorite_color} });
    for my $favorite_color_collection ($collections_by_favorite_color->each) {
      say $favorite_color_collection->[0]->{favorite_color};

      for my $person ($favorite_color_collection->each) {
        say "\t$person->{name}";
      }
    }

    # output is
    # blue
    #     Bob
    #     Alice
    # green
    #     Eve

    # collect by multiple keys
    # uses favorite_color and age as the keys
    my $collections_by_favorite_color_and_age = $c->collect_by(sub { $_->{favorite_color}, $_->{age} });

["collect\_by"](#collect_by) allows you to transform a [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection) into a [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection) of [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection)s,
where all elements of the inner collections share the same single key or list of keys.
["get\_values"](#get_values1) allows you to control which values are collected in the [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection).

### get\_keys

["get\_keys"](#get_keys2) is required and must return a single key or a list of keys. The return value of ["get\_keys"](#get_keys2) will be the keys used group
values that are returned by ["get\_values"](#get_values1) into each inner [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection).

    # return single key
    my $collections = $c->hashify(sub { $_->{name} });

    # return multiple keys
    my $collections = $c->hashify(sub { $_->{name}, $_->{age} });

The current element is available via `$_`, or as the first argument to ["get\_keys"](#get_keys2).

### get\_values

["get\_values"](#get_values1) may return one or more values as a list for the current element in the [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection).

    # return single value
    my $collections_of_age = $c->hashify(sub { $_->{name} }, sub { $_->{age} });

    # i.e.
    # [ [23], [24], [27] ]

    # return multiple values
    my $collections_of_age_and_favorite_color = $c->hashify(sub { $_->{name} }, sub { $_->{age}, $_->{favorite_color} });

    # i.e.
    # [ [23, 'blue'], [24, 'blue'], [27, 'green'] ]

The default is to return the current element:

    # default get_value
    sub { $_ }

    # not passing in get_value uses the above subroutine to return the current collection element, in this case a hash
    my $collections = $c->hashify(sub { $_->{name} });

    # i.e.
    # [ [{ name => 'Bob', ...}], [{ name => 'Alice', ...}], [{name => 'Eve', ...}] ]

The current element is available via `$_`, or as the first argument to ["get\_values"](#get_values1).

### OPTIONS

#### flatten

    # trivial example where returned values are wrapped in arrayrefs to demonstrate flatten
    my $collections = $c->hashify({flatten => 1}, sub { 'key' }, sub { [$_->{age}] });

    # i.e.
    # [ [23, 24, 27] ]

The ["flatten"](#flatten1) option flattens each inner [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection), meaning that
it flattens nested collections/arrays recursively and creates a new collection with all elements. See ["flatten" in Mojo::Collection](https://metacpan.org/pod/Mojo::Collection#flatten)
for more details.

Internally, ["flatten"](#flatten1) is implemented differently for performance, but the end result is the same.

# AUTHOR

Adam Hopkins <srchulo@cpan.org>

# COPYRIGHT

Copyright 2019- Adam Hopkins

# LICENSE

This library is free software; you can redistribute it and/or modify
it under the same terms as Perl itself.

# SEE ALSO

- [Mojo::Collection](https://metacpan.org/pod/Mojo::Collection)
- ["with\_roles" in Mojo::Base](https://metacpan.org/pod/Mojo::Base#with_roles)
- [Role::Tiny](https://metacpan.org/pod/Role::Tiny)
