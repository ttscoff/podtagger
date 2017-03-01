# PodTagger: Automated ID3 tagging for podcasts

<img style="float:right;margin:10px 0 10px;" src="https://github.com/ttscoff/podtagger/raw/master/podtagger.png">

<http://brettterpstra.com/projects/podtagger>

PodTagger is a ruby script that reads a `shownotes.raw` (Markdown with YAML headers) file and applies the information in the headers using configured templates to a target MP3 file.

### Requirements

PodTagger requires [mid3v2](https://mutagen.readthedocs.io/en/latest/man/mid3v2.html) from the Python mutagen package. You can install it with [pip](https://pip.pypa.io/en/stable/). If you don't already have pip installed, use `sudo easy_install pip` before running `sudo pip install mutagen`.

### Configuration

PodTagger reads a YAML config file with defaults, and YAML headers on podcast show notes. Start by creating a default configuration file in your home directory (~) called `podtagger.yaml`. If you run it without creating this file first, it will automatically create an empty config for you.

#### podtagger.yaml

The podcast file contains the following keys:

```yaml
default:
  # Title of podcast
  podcast: PODCAST NAME
  
  # Name of host(s)
  host: HOST NAME 
  
  # Podcast network name, blank if not applicable
  network: NETWORK 
  
  # Format for adding header to output show notes
  # Any meta key can be used within %% variables
  title_format: "%%title%% with %%guest%%" 
  
  # Format for title added as ID3 tag
  # Any meta key can be used within %% variables
  ep_title_format: "%%title%% with %%guest%% - %%podcast%% %%episode%%"
  
  # POSIX path to thumbnail image
  logo: PATH/TO/THUMBNAIL 

## Optionally add configs for podcast-specific keys
# PODCAST NAME:
#   ep_title_format: "%%title%% with %%guest%% - %%podcast%% %%episode%%"
#   title_format: "## %%title%% with %%guest%%"
#   logo: PATH/TO/THUMBNAIL
```

Adding additional top-level keys with podcast names allows you to override settings on a per-podcast basis. If you only ever tag one podcast, you can just use the `default:` key/values.

If you have additional podcast configurations, they extend the default, meaning that keys that aren't defined in that podcast section fall back to whatever is defined in `default:`. Thus, you only have to add keys for settings you want to override.

As an example, here's my `podtagger.yaml` for Overtired and Systematic.

```yaml
default:
  podcast: Systematic
  title_format: "## %%title%%"
  ep_title_format: "%%title%% - %%podcast%% #%%episode%%"
  network: ESN.fm
  host: Brett Terpstra
Systematic:
  ep_title_format: "%%title%% with %%guest%% - %%podcast%% %%episode%%"
  title_format: "## %%title%% with %%guest%%"
  logo: /Users/ttscoff/Desktop/Systematic-esn-thumb.jpg
Overtired:
  host: Christina Warren & Brett Terpstra
  ep_title_format: "%%title%% - %%podcast%% #%%episode%%"
  logo: ~/Desktop/overtired-thumb.jpg
```

As you can see, the Systematic values only override the title formats (see "Format strings" below) and add a logo. The Overtired values override the host tag, the logo, and the episode title format (the one used on the MP3, not in the show notes).

Any of these values can be overridden by the YAML data in an episodes notes file.

#### Format strings

The keys `ep_title_format` and `title_format` use variables delineated by 2 percent symbols on either side: `%%variable%%`. The variable name can correspond to any keys defined, such as `%%podcast%%` or `%%title%%`. You can also include custom keys in your `shownotes.raw` headers (see below) which are then available as variables. For example, my Systematic show notes include a key for `guest:`, which I can then use in the format string: `"## %%title%% with %%guest%%"`.

### Show Notes

PodTagger reads a YAML header from a file called `shownotes.raw` in the same directory as the mp3 file you're tagging.[^1] After PodTagger finishes, it will write out the show notes with the YAML removed to a file called `shownotes.md`. The rest of the file after the YAML headers can be any information you want. I use it for description, sponsor info, and show links, but it's for whatever you'd post on a show landing page.

The `shownotes.raw` file would look something like this:

```
---
description: "Lawyer, blogger, podcaster, and Mac productivity guy, David Sparks joins Brett to talk about freelancing, saying no to projects, and the latest and greatest top 3 picks."
episode: 185
guest: David Sparks
---
# Too Busy

This episode is sponsored by TextExpander for Teams, making all of your team's common replies consistent and easily accessible. Learn more (and support Systematic) at [textexpander.com/systematic](https://textexpander.com/systematic?utm_source=systematic&utm_medium=podcast&utm_campaign=textexpander-feb17
).
...
```

Note that you can include a `title:` meta key, or just make an h1 or h2 (`# title` or `## title` line) in the show note body. Either will generate the title and format it based on format strings in the configuration.

[^1]: If `shownotes.raw` doesn't exist but `shownotes.md` does, `shownotes.md` will be copied to .raw and then overwritten without the YAML headers.
