# automod
auto-moderation system for (r/)music

## Overview
This is a demonstration of the auto-moderator system I helped overhaul for a huge online community (>30M users) that I moderate. It was implemented in 2018 and is updated regularly. Reddit's automoderator configs are written in the data-serialization language [YAML](https://yaml.org/). I've redacted some of the regex statements critical to our filter in order to prevent bad-faith usage.

## Key Problems
We designed this content filtering system to address a few critical oversights that were causing a massive influx of spam and rule-breaking content:
- Self promotion from new accounts with no previous community interaction
- Hate speech and political trolling
- Mass brigading from other communities
- Piracy and copyright infringement
- Persistent reposts from the list of banned "Hall of Fame" artists

## Design
Here's how we tackled these issues:
### Promotional Spam and Piracy
 - Recognized that self-promotion violations have an extremely high correlation with account age, low comment interactions, and certain keywords representing unsolicited "promotional events".
    ```
    type: submission
    action: remove
    ~title (regex): ["... "ask ?(me|us)? ?(almost)? anything", ... (redacted)"]
    ~flair_css_class: "amas"
    author:
        account_age: "< 1"
        is_contributor: false
        is_moderator: false
    ```
- Automated the addition of new spam and piracy identifying tokens to the YAML config.

		type: submission
		domain+url (includes): [newsroniaan.com,
		1833.fm,
		1888pressrelease.com,
		24ourmusic.net, 
		2dyr.com,
		...
		]
		action: spam

	And the associated Python extract:
		
		add_to_spam(domain):
			if domain not in spam_list:
				spam_list.append(domain)
			return

### Brigading and Bad-faith Political Discourse
- Trained the offline filter to recognize tokens that incite brigading from other subreddits and high levels of community reports.

	```
    type: submission
    title+body (includes, regex): ["1488", "trump", "lib" ... "]
	```
		
- But hate groups have gotten significantly more adept at bypassing basic filters through the use of coded linguistic signalling and dog-whistles. So we also explicitly include known quantities of this sort:

		type: any
    	body+title (includes): ["((("]
    	action: filter
    	report_reason: "Review for anti-semitic '(((tagging)))' and brigading - {{match}} - https://en.wikipedia.org/wiki/Triple_parentheses"
			
### Repetition
Being a "default" subreddit for many years means that the usership of our sub is a lot more general than most niche communities, so it makes sense to limit the number of posts linking to old music by "iconic" artists such as Daft Punk and Radiohead. For now, we just explicitly program this into the filter with a publicly available list of "Hall of Fame" artists.

	type: submission
    title (regex, includes): ["\\b(arcade fire|arctic monkeys|beastie boys|beatles|blink 182|cake|childish gambino|daft punk| ...
		..."]
