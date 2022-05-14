# hugo-toroidal

Tooling for hosting your own webring using a hugo site/repo as the source of truth/host.
Inspired by [TooManyBees/ring](https://toomanybees.github.io/ring/about).

## Installing Toroidal to Your Hugo Site

This is a hugo module.
You'll need to [make sure your site is ready to add this module to it](https://gohugo.io/hugo-modules/use-modules/).

Once that is done, you can update your site config to reference it:

```yaml
#config.yaml
module:
  imports:
    - path: 'github.com/platenio/hugo-toroidal'
```

```toml
#config.toml
[module]
[[module.imports]]
  path = 'github.com/platenio/hugo-toroidal'
```

Once you've saved your updated config file, you can run `hugo mod get -u` to pull this module down for use.

### Start From Template

Top of our todo list is to set up an extremely minimal default site for you to use which is ready to go.
We'll update this section when that's ready!

## Configuring Your Webring

Right now there's only a few configuration items for this module and they're optional.

### Webring Name

If you'd like to name your Webring, edit your site config:

```yaml
#config.yaml
params:
  Toroidal:
    WebringName: 'FooBar'
```

```toml
#config.toml
[params]
  [params.Toroidal]
  WebringName = "FooBar"
```

The above example will change the header for your site member list to `Members of the FooBar Webring` and the generated webring navigation label to `FooBar Webring`.

If left unset, the header for your site member list will read `Members of the Webring` and the generated webring navigation label will read `Webring`.

### Random Member Link

By default, the iframes for your webring include the name of the webring and then a nav list enabling users to go to the previous member of the webring,
the list of all webring members, or the next member of the webring.

With the `RandomMemberLink` configuration option, you can enable users to click a link to navigate to the homepage of a random member of the webring.

```yaml
#config.yaml
params:
  Toroidal:
    RandomMemberLink: true
```

```toml
#config.toml
[params]
  [params.Toroidal]
  RandomMemberLink = true
```

Specifying the value for this configuration item as `false` is the same as not specifying it.
You must opt in by setting this configuration item to `true` to enable the random member link.

### Theming

Minimal theming has been added to the member pages of the webring. This theme uses SCSS in the
`assets/toroidal` folder. By default, the theme is set to **Auto**, which chooses between the
built-in light and dark themes depending on the user's operating system setting preferences.

You can specify an override via the `Theme` configuration option:

```yaml
#config.yaml
params:
  Toroidal:
    Theme: custom # or light, dark, or auto
```

```toml
#config.toml
[params]
  [params.Toroidal]
  Theme = 'custom' # or light, dark, or auto
```

The `custom` option allows you to design your own theme from scratch; to do so, you will need to
shadow the `assets/toroidal/_custom.scss` file in your own site repo.

You can also override any variables you choose by shadowing the `assets/toroidal/_variables.scss`
file. In the future, we'll have documentation explaining all of the variables. For now,
unfortunately, you need to [check out the source][source-variables] to see what is in there.

If you have any ideas for theming or run into anything you'd like us to make more extensible,
please [file an issue][file-an-issue].

## Adding Members

You can add new member sites to your webring by adding a markdown file in your `content/toroidal` folder.
The only thing you need to include in that file is some frontmatter to tell hugo about the site.
For example:

```yaml
# content/toroidal/first-member.md
---
name: First Member
homepage: https://firstmember.com
description: The first member's site
weight: 1
---
```

This will:

- Set the display name of the site in the member list to `First Member`
- Set all links to that site (on the member list and in the generated webring navigation) to `https://firstmember.com`
- Set the short description of the site in the member list to `The first member's site`
- Set their member number to 1; this is used both to display the members in the correct order in the member list and to determine which sites are considered "previous" and "next" in the generated webring navigation.

You will need a file for every member site added to the webring.
For now, we don't have any helpers to ease this, but we're going to write some tooling and update this doc as those land.

## Adding the Webring Navigation to Your Member Site

To add the webring navigation to your member site, you'll need to:

1. Have your member page added to the webring host site and live on the internet.
2. Know the name of the file added (we suggest using your site name lowercased with hyphens instead of spaces; so "My Really Neat Site" would become `my-really-neat-site`)
3. Add an iframe to your site where you want the navigation to go:
   ```html
   <iframe src="https://superneatwebring.com/toroidal/my-really-neat-site">
   </iframe>
   ```

That will add the navigation menu wherever you placed it in your own site code.
If you inspect the iframe, you'll see HTML like this:

```html
<nav aria-labelledby="toroidal-menu-label">
  <h2 id="toroidal-menu-label">Super Neat Webring</h2>
  <ul role="menu" id="toroidal-menu" aria-label="Super Neat Webring">
    <li role="none">
      <a role="menuitem"
         aria-haspopup="false"
         target="_parent"
         href="https://previous-member.com">Previous</a>
    </li>
    <li role="none">
      <a role="menuitem"
         aria-haspopup="false"
         target="_parent"
         href="http://superneatwebring.com/toroidal/">Member List</a>
    </li>
    <li role="none">
      <a role="menuitem"
         aria-haspopup="false"
         target="_parent"
         href="https://next-member.com">Next</a>
    </li>
  </ul>
</nav>
```

> Of course, the URLs and webring name will match your webring!

### Theming

By default, this module does _zero_ special theming for either the member site list or the generated webring navigation menu.
Your member site list will inherit your hugo theme's defaults and display a paginated list of sites (10 per page, and users can click through the list to see more members if needed).

Your webring navigation will be pure unstyled HTML in an iframe.

Some default themes will be available in the future.
For now, here's some info you can use to theme things for yourself:

1. The member list is inside of an `article` tag with the `toroidal` class.
   1. It includes an `h2` tag with the text "Members of the `Webring Name` Webring"
   2. It includes a `ul` tag with the `toroidal-member-list` class
   3. Each of the member sites in that list are in a `li` tag with the `toroidal-member-entry` class
   4. Each `li` includes an `a` tag with their site name and referencing their URL followed by a `span` for their description
2. The generated webring navigation is inside of a `nav` tag with the `toroidal` class
   1. The `h2` that labels the navigation has the id `toroidal-menu`
   2. Each entry in the menu is an `li` tag with role set to `none` with an `a` tag whose role is set to `menuitem`

## Contacting Us

If you have any questions, comments, or concerns, you can reach out to us by
[starting a discussion][start-a-discussion], [filing an issue][file-an-issue], or reaching out to
the maintainers on Twitter or Discord.

[file-an-issue]: https://github.com/platenio/hugo-toroidal/issues/new
[start-a-discussion]: https://github.com/platenio/hugo-toroidal/discussions/new