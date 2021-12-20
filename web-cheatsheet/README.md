# Web cheatsheet

A collection of notes for everything related to web pentesting.

## Requests that don't care about CORS

Always look for missing CSRF tokens no matter if CORS is well-implemented or not. For instance, the following requests don't care about CORS:

* Loading an image via an `img` tag
* Sending a `POST` request by clicking the submit button of an HTML `form` element

CORS doesn't shield you from all cross-origin requests, plain and simple.

## Mix things up

* Send GET requests that were meant to be POSTed, and vice versa
* Change the content-type in requests and see if the target still accepts it:
  * Example here, using `text/plain` with JSON endpoints: https://blog.azuki.vip/csrf/

## Don't forget about some JS weaknesses

* JS tries to evaluate everything, so inserting a payload like this works: `'string'-alert(1)-'string'`

## Tilde convention for backups

File names with a tilde appended (like_this.txt~) are a convention for backups. Be sure to check them out if you find a browsable file somewhere.
