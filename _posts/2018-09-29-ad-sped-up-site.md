---
layout: post
title:  How an ad sped up our site
date: 2018-09-29 19:00:00
---

_This post was first published on the Hipmunk engineering [blog](https://hipmunk.github.io/posts/2018/Sep/29/how-an-ad-sped-up-our-site/)._


# New homepage

Advertising is a big source of revenue for Hipmunk. We are working to make sure the ads we display are relevant and unintrusive.
We recently launched a new ad placement on our homepage. This ad is powered by the Google Ad Manager which we also leveraged in an unusual way and allowed us to make our site faster.

![Homepage Ad unit](/img/2018-09-29/home_rover.png)

## Requirements

No page is more important for the identity/branding of a web product than the homepage. The idea to add an advertisement on the Hipmunk homepage was first met with a lot of resistance internally. After a lot of discussions, we came up with a solution that would both monetize in some way our homepage but also keep a strict control over the look and feel of these advertisements.

Our design team was already creating custom background images for our homepage from time to time.

![Flower Power](/img/2018-09-29/flower.jpeg)
![Chippy in the snow](/img/2018-09-29/snow.png)

For this new ad, we would continue to design custom backgrounds, but with a theme close to the product/company buying the placement. In addition, we will use a subtle call to action to drive users to their product.

![Example of background ad image](/img/2018-09-29/roam.jpg)

This call to action has to be powered by the Google Ad Manager (ex DoubleClick for Publisher) so our Ads team can manage it and we can monitor the performance on this unit.

This advertisement and its custom background image need to be visible only to some of our users.

Finally, we want to keep our homepage as a "static" HTML that gets generated only once per deploy and is then cached in our CDN.

## What was existing

As mentioned before, we were already deploying from time to time new hero images. This was a manual process, requiring custom code for each image, which means an engineer always had to be involved.

Because each image had different colors and layout, we had to tweak styles for each image.

```scss
        &.weeks_52 {
            background-repeat-x: repeat;
            background-size: auto;
            background-position: center center;
            background-image: linear-gradient(
                to bottom,
                rgba(0, 0, 0, 0.1) 0%,
                rgba(0, 0, 0, 0.2) 100%
            );

            &.weeks_52_autumn {
                background-image: none;

                .FrontpageBase__slogan {
                    color: #685642;
                }

                .FrontpageBase__partner-logos-container {
                    color: #554533;
                }
            }
            &.weeks_52_winter {
                background-image: none; // removes linear-gradient

                .FrontpageBase__partner-logos-container {
                    color: #153f5e;
                }
            }
            &.weeks_52_valentines {
                background-image: none; // removes linear-gradient
                .FrontpageBase__slogan {
                    opacity: 0;
                }

                .FrontpageBase__partner-logos-container {
                    color: #153f5e;
                }
            }
```

As you can see in the snippet of SCSS, we were keeping older images in the code. We were leveraging our feature flag infrastructure to control which image to display. This allowed us to prepare to display a header beforehand and releasing it into the wild with a couple of clicks.

While working, this system always requires an engineer to do some code change and a deploy, and some back and forth with a designer to adjust the image.

I am sure you are thinking that a simple solution would be to build a small internal GUI to control all of these. While great in theory, in practice, it would be against our pragmatic philosophy. The time it took an engineer to add a new background image was fairly small (let's say 20min) and happening a few times a year. So before we spend several workdays building a tool that fits all our need, this task has to happen way more often that it becomes a real burden on us. That's the whole idea behind the expression "do things that do not scale" so dear to startups.

## Using Google Ad Manager

With the creation of this homepage unit, the situation changed. The image should change more often, and we would need to control more precisely when to show it: ad campaigns are usually negotiated to run between specific date or for a specific number of impressions.

As mentioned in the requirements for this new homepage ad, we will use the Google Ad Manager to control what call to action to display.

Google Ad Manager allow someone to create ad campaigns, control who, when and how many people see it. The tool even allows you to create your own ad. It provides a UI to upload images, configure the campaigns in any way you can imagine.
This seems like an exact copy of what our custom background image GUI tool should do. So much, that we decided to configure and display our background images as an ad.
By leveraging the Google Ad product, we did not have to build/maintain anything outside of our core product, but we also have been to collaborate very closely with our commerce team. They were the one advising us how to leverage the tools to save everybody time. Historically, engineer and ads people can see the world with very different perspective, but this project helps us align.

The ad output is a JSON object that contains some of the settings as well as the URL of the image to use as a background image. It is loaded by our JavaScript which then configures the background image.

## Ad blocker workaround

While saving us time on the back-office part, this new ad unit/background image had some issue:

- it requires the google ads library to be loaded before we can display the image
- it will degrade the experience for people using an ad blocker
- it should respect cookie privacy setting of the user

The cookie issue is the most important problem. Hipmunk is GDPR compliant, and it means respecting the user choice regarding cookies. In order to do so in the browser, we would need to wait for our code and an external library to be loaded and executed in order to check if we can start to load DFP. All this would delay for a significant amount of time the display of the hero image.
In addition, if the user does not accept advertisement cookies or use an ad block, the homepage will look less engaging and fun.

![Homepage, without a background image](/img/2018-09-29/no_back.png)

We love Chippy and we want to show him to the world! So, we built a proxy service that loads the ad configuration from our server and caches it for a few minutes. By using a proxy, we do not set any cookies, and we do not need to wait for our JavaScript to start loading the background image.

## Faster user loading

One of the features we offer to our [Concur Hipmunk](https://www.hipmunk.com/concur) customers is the ability to use Hipmunk without any advertisements. This means we do not want to show this homepage unit for these customers.

Our frontpage is a mostly static HTML page that gets generated once after each deploy and then is cached in our CDN. Any information about the user is loaded by an AJAX request. The problem with that is that is happening way too late.

![Loading the user before](/img/2018-09-29/load_before.png)

Before the AJAX request is even starting, the browser needs to load the HTML of the homepage, download our JavaScript, parse it and finally execute it. While we are always trying to make these steps faster, it still takes too long. The homepage background image is showing up way too late, creating a weird flash.

Our solution to this problem was to inline in the HTML of the homepage some code making the API call to load the user information. Being inline, we by-pass completely the time it takes to load, parse and execute our JavaScript bundles.

![Loading the user after](/img/2018-09-29/load_after.png)

This optimization is making our site feel snappier when loading and even is helping speeding up our search under some conditions.

## Conclusion

With this post, I hope to have demonstrated that Hipmunk employees' obsession to deliver a great UX to our user allows us to find creative solutions to problems. It is also a great example that every project is an opportunity to make your application better -- whether it's visible to users or something behind the scenes.
