# Verify HTTP links with awesome_bot

When writing articles, blog posts, technical papers, README's etc, it's common to reference other web pages. Your content may not move, but there is no guarantee other peoples stuff won't - sites go down, links change and certificates expires. Nothing can be as frustrating as finding that 6 year old blog post with the answers to your problems, but then find it full of dead links. Thankfully you can set a good example and check your own content for dead links with a little Ruby-tool called [awesome_bot](https://github.com/dkhamsing/awesome_bot).

The `awesome_bot` name comes from the "awesome pages" that are rather popular on GitHub. If you don't know about Awesome, it's usually collections of links with "awesome" tools, resources, libraries etc. for a specific topic or ecosystem. Examples are [awesome-dotnet](https://github.com/quozd/awesome-dotnet), [awesome-react](https://github.com/enaqx/awesome-react), [awesome-ai-awesomeness](https://github.com/amusi/awesome-ai-awesomeness) or [awesome-ci](https://github.com/ligurio/awesome-ci) - the list is endless. Needless to say, these pages contain lots and lots of links, hence the need for validating the aliveness of these links.

If you are into Ruby and gems, `awesome_bot` can be installed with `gem`:

```shell
gem install awesome_bot
```

I prefer the container approach, and [dkhamsing/awesome_bot image can be used](https://github.com/dkhamsing/awesome_bot#docker-examples).

An example execution of `awesome_bot` might look like this:

```shell
$ docker run --rm -v $(pwd):/mnt -t andmos/awesome-bot -f **/*.csv --allow-redirect --allow 429
> Checking links in misc/Roasteries.csv
> Will allow errors: 429
> Will allow redirects
Links to check: 14
  01. https://www.kaffebrenneriet.no/
  02. https://www.timwendelboe.no/
  03. https://sh.no/
  04. https://www.kaffa.no/
  05. https://www.srw.no/
  06. https://www.fjellbrent.no/
  07. https://jacobsensvart.no/
  08. https://www.facebook.com/stormkaffe
  09. https://www.pala.no/
  10. https://www.langorakaffe.no/
  11. https://inderoy.coffee/
  12. https://bonneribyen.no/
  13. https://www.facebook.com/brentkaffe/
  14. https://senjaroasters.com/
Checking URLs: ✓✓✓✓✓✓✓→✓✓✓✓✓✓
No issues :-)
```

In this example we check links from CSV files and use the flags `--allow-redirect` to, well, allow redirects (which throws errors if not given) and `--allow 429` to whitelist the "Too many requests" status code.

If something is off with a link, like a 404, `awesome_bot` will throw an exit-code and show issues in the report:

```shell
$ docker run --rm -v $(pwd):/mnt andmos/awesome-bot -f *.md
> Checking links in README.md
Links to check: 1
  1. https://www.an.no/some/dead/link
Checking URLs: x

Issues :-(
> Links
  1. [L1] 404 https://www.an.no/some/dead/link
> Dupes
  None ✓

Wrote results to ab-results-README.md.json
Wrote filtered results to ab-results-README.md-filtered.json
Wrote markdown table results to ab-results-README.md-markdown-table.json
```

`awesome_bot` can be automated to run scheduled with you favorite CI system - here is a GitHub Actions example:

```yaml
name: Verify Links
on: 
  pull_request:
  workflow_dispatch:
  schedule:
    - cron:  '0 13 * * 1'
jobs:  
  Awesome-bot:
    name: Run Awesome-bot
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v3

      - name: Verify Links
        run: |
          docker run --rm -v $(pwd):/mnt andmos/awesome-bot -f *.md --allow-redirect --allow 429 --allow-ssl --white-list "nasdaq.com,researchgate.net"
```
