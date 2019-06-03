# NAME

Method::Delegation - Easily delegate methods to another object

# VERSION

Version 0.01

# SYNOPSIS

    package Order;

    use Method::Delegation;
    delegate(
        methods => [qw/name customer_number/],
        to      => 'customer',
    );

# EXPORT

## delegate

Calling `delegate(%args)` installs one or more methods into the current
package's namespace. These methods will delegate to the object returned by
`to`.

Arguments to `delegate` are as follows (examples will be after):

- `to`

    This is the name of the method that, when called, returns the object we
    delegate to.  It is assumed that it will _always_ return an object, unless
    the _if\_true_ argument is also supplied.

- `methods`

    These are the names of the methods we can call and what they can delegate to.
    If a scalar is supplied, this will be the name of the method of both the
    calling package and the delegated package.

    If an array refererence, this will contain a list of names of the method of
    the both the calling package and the delegated package.

    If a hash reference, the keys will be the methods of the calling package and
    the values will be the names of the method of the delegated package.

- `args` (optional)

    By default, we assume that these delegations are read-only. If you pass
    `args` and give it a true value, the method created in the delegation will
    attempt to pass your args to the method we're delegating to.

- `if_true` (optional)

    The name of a method to call to see if we can perform the delegation. If it
    returns false, we return false. Usually this is the same name as the `to`
    argument, meaning if if the method named in `to` does not return an object,
    simply return false instead of attempting to delegate.

- `else_return` (optional)

    If `if_true` is supplied, you may also provide `else_return`. This must
    point to a scalar value that will be returned if `if_true` is false.

- `override`

    By default, we will not install a delegated method if the package already has
    a method of that name. By providing a true value to `override`, we will
    install these methods anyway.

# EXAMPLES

## Basic Usage

Delegating a single method:

    delegate(
        methods => 'name',
        to      => 'customer',
    );
    # equivalent to:
    sub name {
        my $self = shift;
        return $self->customer->name;
    }

Delegating several methods:

    delegate(
        methods => [/name rank serial_number/],
        to      => 'soldier',
    );
    # equivalent to:
    sub name {
        my $self = shift;
        return $self->soldier->name;
    }
    sub rank {
        my $self = shift;
        return $self->soldier->rank;
    }
    sub serial_number {
        my $self = shift;
        return $self->soldier->serial_number;
    }

Delegating, but renaming a method:

    delegate(
        methods => {
            customer_name   => 'name',
            customer_number => 'number,
        },
        to => 'customer',
    );
    # equivalent to:
    sub customer_name {
        my $self = shift;
        return $self->customer->name;
    }
    sub customer_number {
        my $self = shift;
        return $self->customer->number;
    }

## Advanced Usage

Delegating to an object that might not exist is sometimes necessary. For
example, in [DBIx::Class](https://metacpan.org/pod/DBIx::Class), you might have an atttribute pointing to a
relationship with no corresponding entry in another table. Use `if_true` for
that:

    delegate(
        methods => 'current_ship',
        to      => 'character_ship',
        if_true => 'character_ship',
    );
    # equivalent to:
    sub current_ship {
        my $self = shift;
        if ( $self->character_ship ) {
            return $self->character_ship;
        }
        return;
    }

Note: the `if_true` attribute doesn't need to point to the same method, but
usually it does. If it points to another method, it simply checks the truth
value of the response to determine if the original delegate will be called.

Sometimes, if the object you're delegating to another object, you want a
different value than `undef` being returned if the object isn't there. Use
the `else_return` attribute. The following will return `0` (zero) instead of
undef if `current_weapon` isn't found:

    delegate(
        methods => {
            weapon_damage   => 'damage',
            weapon_accurace => 'accuracy',
        },
        to          => 'current_weapon',
        if_true     => 'current_weapon',
        else_return => 0,
    );

# AUTHOR

Curtis "Ovid" Poe, `<curtis.poe at gmail.com>`

# BUGS

Please report any bugs or feature requests to `bug-method-delegation at
rt.cpan.org`, or through the web interface at
[https://rt.cpan.org/NoAuth/ReportBug.html?Queue=Method-Delegation](https://rt.cpan.org/NoAuth/ReportBug.html?Queue=Method-Delegation).  I will
be notified, and then you'll automatically be notified of progress on your bug
as I make changes.

# SUPPORT

You can find documentation for this module with the perldoc command.

    perldoc Method::Delegation

You can also look for information at:

- RT: CPAN's request tracker (report bugs here)

    [https://rt.cpan.org/NoAuth/Bugs.html?Dist=Method-Delegation](https://rt.cpan.org/NoAuth/Bugs.html?Dist=Method-Delegation)

- CPAN Ratings

    [https://cpanratings.perl.org/d/Method-Delegation](https://cpanratings.perl.org/d/Method-Delegation)

- Search CPAN

    [https://metacpan.org/release/Method-Delegation](https://metacpan.org/release/Method-Delegation)

# ACKNOWLEDGEMENTS

# LICENSE AND COPYRIGHT

This software is Copyright (c) 2019 by Curtis "Ovid" Poe.

This is free software, licensed under:

    The Artistic License 2.0 (GPL Compatible)
