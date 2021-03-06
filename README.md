# Morphoji

[![Build Status](https://travis-ci.org/chefkoch-dev/morphoji.svg?branch=master)](https://travis-ci.org/chefkoch-dev/morphoji)

Morphoji is a tiny PHP library to **morph** Unicode Em**oji** characters 🤗 into 
Latin1 placeholders🙀 and back. 👍

## Use Case

Why would you want to do this in the first place? Maybe for the same reason I
did: you got a big old MySQL Database with all text columns defined as `utf8`.
Great, because with `utf8` you can store everything Unicode has to offer, 
including Emoji characters, right?

Wrong-o.

Apparently `utf8` in MySQL (and some other applications) is limited to [3 Byte 
Unicode characters](https://en.wikipedia.org/wiki/UTF-8#Description). 
Unfortunately the bulk of the 
[Emoji characters](https://unicode-table.com/en/#emoticons) is found in 4 Byte
Unicode space.

Trying to store those Emoji (and other 4 Byte characters) in a MySQL `utf8`
table will result in those characters silently getting lost.

### utf8mb4

Of course there is a systematic solution for this. Convert the columns in 
question (and your database connections to them) from `utf8` to `utf8mb4`. THAT
charset is actually able to store ALL of the characters specified by Unicode.

If you want to go that route (which is by far the cleanest approach) Mathias 
Bynens wrote a 
[great article on that](https://mathiasbynens.be/notes/mysql-utf8mb4).

But also, if your database is really big and has lots of text columns and you
don't want to convert just a few columns to the new charset but all of them
(because consistency and because you will have to change your connection's
charset as well) and you really only care about Emoji and not about 
characters for 
[Ancient Greek Musical Notation](https://unicode-table.com/en/#ancient-greek-musical-notation) 
aaand you just did all that converting a couple of years ago for `utf8` and 
don't really have the time and nerve to do it again ...

... you could just use Morphoji. 

## Usage

Morphoji can be required via [Composer](https://getcomposer.org) in the project
where you want to use it:

```bash
composer require chefkoch/morphoji
```

### Converting

Now if you have `$text` containing (possibly) Emoji characters handle it like 
this:

```php
$text = '...'; // Some text with unicode emojis.


$converter = new \Chefkoch\Morphoji\Converter();

// From utf-8 text to db.
$textForDb = $converter->fromEmojis($text);
$db->insert($textForDb); // Dummy code for DB insert command.

// From db to utf-8 text.
$textWithEmoji = $converter->toEmojis($db->getTextWithPlaceholders());
return new Response($textWithEmoji); // Dummy code for HTML response to browser.
```

### Wrapping

Additionally you can convert a string with emoji placeholders to a string where
these placeholders are wrapped with arbitray prefix and suffix.

For example you can use this to decorate the placeholders with tags.

```php
$placeholderText = 'Lorem :emoji-12345: ipsum';

$converter = new \Chefkoch\Morphoji\Converter();

$wrapped = $converter->wrap($placeholderText, '&nbsp;');

// results in $wrapped == 'Lorem <span class="emoji" data-emoji-code="12345">&nbsp;</span> ipsum'
```

This way you can replace them with your own graphics when rendering the text in 
whatever your frontend may be.

## How it works

Morphoji will only convert characters which have an official emoji 
representation. This happens by using a regular expression derived from 
[official Unicode charts](http://www.unicode.org/Public/emoji/5.0/).

Every character matching that regular expression will be converted into a
latin1 placeholder:

```
:emoji-[hex code]:
```

E.g. a replaced "face blowing a kiss" Emoji (`1F618`, 😘) will be represented 
as `:emoji-1f618:`.

Converting the placeholder back works with an according regex; other than that 
it's pretty much vice versa.

### Why not just convert into HTML entities and then use those in the output?

Morphoji aims to be output device agnostic. If you just want to HTML everything
that's fine, but in that case this is probably not the library you are looking
for.

In my case the data stored in the database isn't necessarily output in an HTML
context, so being able to convert the entities back is mandatory. (And in a time
where almost all output devices / applications are able to handle full UTF-8 it
is the generally cleaner approach.)

## Other languages

There are no plans to implement this in any other language. Coming up with the
emoji detection regex is about half the work, if you want to use it in your own 
implementation: go ahead.

## Tests

If you want to contribute, please don't forget to add / adapt the tests.

To run them:

```bash
composer install
vendor/bin/phpunit
```

## License

Morphoji is licensed under the [MIT License](LICENSE).
