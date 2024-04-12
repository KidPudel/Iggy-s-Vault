[Info from](https://www.zenrows.com/blog/selenium-avoid-bot-detection)
Ways to trick anti-bot to think that you are not a bot
1. Proxy to pass [[WAF]] (though site can block VPNs)
2. Disabling Automation indicator WebDriver flag, which anti-bot could look for
3. Rotating [[backend/network/headers|headers]] HTTP header contains information, and headless browser have some of differences in attributes, which anti-bot could look for
4. Avoid patterns
5. Remove JavaScript signature stored in `cdc_`

For all that we can use [undetected_chromedriver selenium](https://github.com/ultrafunkamsterdam/undetected-chromedriver)

6. Adding browser profile: To make us even less suspicious add profile path (chrome://version/) to default Options 