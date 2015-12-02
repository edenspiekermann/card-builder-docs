# Setting up server side includes using ESI

In this short documentation, you will learn which ESI endpoints you can use to include Card Builder stacks on your website.

* [ESI Endpoints](#esi-endpoints)
  * [ESI Head](#esi-head)
  * [ESI Body](#esi-body)
  * [ESI Scripts](#esi-scripts)
* [Passing a custom sharing URL](#passing-a-custom-sharing-url)

## ESI Endpoints

To implement card stacks on your site using server side includes please use the following structure. You have a unique card stack base URL, or maybe even more than one if you have a separate advertising account, this will be something like http://yourname.cards.edenspiekermann.com – and all the paths that follow are based on this.

### ESI Head:
```
/stacks/esi/head
```
This contains styles, and sprites. Should be included in the head of the article.
Optional: Add a ‘static=true’ parameter to serve narrow stacks on homepages

#### ESI Head with Meta:
```
/stacks/:id/esi/head
```
This contains styles, and sprites. Should be included in the head of the article. This also contains meta tags for title, image and description. This should be included instead of the head when the article has been shared from the card stack using the shareUrl parameter (more on this later)
Optional: Add a ‘static=true’ parameter to serve narrow stacks on homepages

For each stack you wish to embed, one or more, include:

### ESI Body:
```
/stacks/:id/esi/body
```
The main body of the stack, insert this where the stack should appear in the article.

Optional: Add a ‘static=true’ parameter to serve narrow stacks on homepages

Include this once after your scripts:

### ESI Scripts:
```
/stacks/esi/scripts
```
The bit that makes it work. Include after your js files for performance.

## Passing a custom sharing URL
By default the stack will share the current page it’s being displayed on. This is mostly just a fallback option and isn’t the prefered way of sharing. There should also be a share URL set in the admin area of the stack – this is where the stack lives permanently on the parent site. This is especially useful when the user get’s to the stack by browsing related stacks, and wants to share this specific stack.

As well as this a custom share URL parametercan be added to the Body ESI URL. This can have something like campaign tracking parameters, which should be fired when the stack is shared.
```
http://name.cards.edenspiekermann.com/stacks/:id/esi/body?shareUrl=http://url.com?param=true
```
