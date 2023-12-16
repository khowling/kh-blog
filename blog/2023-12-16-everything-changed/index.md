---
slug: everything-changed
title: Everying's changed. Will I use React again?
tags: [htmx, react, chatGPT]
image: preview.png
authors: [keith]
---

import reactlogo from './logo192.png';
import nextlogo from './nextjs.png';

## Starting a new Project in 2024

<table className="logotable" cellspacing="0" cellpadding="0">
<tr className="logotable">
    <td className="logotable"><img src={reactlogo} style={{width:"150px"}} /></td>
    <td className="logotable"><img src={nextlogo} style={{width:"200px"}} /></td>
    <td className="logotable"><span style={{margin: "0 15px"}} class="vite">Vite</span></td>
    <td className="logotable"><div className="logo">&lt;<b>/</b>&gt; htm<b>x</b></div></td>
</tr>
</table>


Time to build a **ChatGPT** app to learn how to develop solutions for around a new user experiance.  This will be a feed based UI with a form with 1 input fixed at the bottom, the scrolling feed rendering results both from chatGPT and Mongo. Sounds like a SPA

So, let get it, I'll start with my trusty tech stack of course,  I've used for years & it's been delivering!: `React`, `Node/Typescript`, `Mongo`. Recent times I've added `tRPC`



:::tip
	I like making the server/browser interface as simple as possible, in fact I'm a big advocate of "Complexity Kills", so I try not to abstract, inject, configure unless necessary.  Hence I've stayed clear of graphQL & I think that’s been a good call.
:::

## Everything has Changed

I look into recent release of React to iterate my stack with the latest guidance from Facebook, the last big update was hooks, and I loved them, no more classes!  

:::tip
Im a fan of the OSS team at Facebook, I see so much great tech coming from them.  I started using client frameworks with Angular1.x back in the day, as I thought you can rely on google for webdev, oh my god, what a mess, I spent 80% of the time understand the framework, and 20% building my app, then the Angular1 ->2, ugh.  I haven't used any google OSS projects since (aside from Kubernetes now in CNCF)
:::

**Everything has changed!**  `create-react-app` is no longer recommended, looks like everyone is using **[Vite](https://vitejs.dev/)** for SPAs, and also React is changing from a client side framework to a server side framework


### <span class="vite small" >Vite</span>

Ok, [Vite](https://vitejs.dev/) it is! It actually turned out to be a great drop-in replacement for `create-react-app`, I don’t need to change my workflow, and everything is faster.  So let's go!

2 days later, I realise this is no longer great choice for my app. Although it’s a SPA, I have hit 2 big issues:

	* The input needs to be POSTed to the server, and the resulting server message needs to be appended to the scrolling feed, I guess I will `useState` hook with an array of messages for the feed items,  this will only ever get appended to, feels like this will slow down as this gets bigger and bigger.  So do I useEffect for my main UI component, essentially bypassing react making it a leaky abstraction?
	* tRPC doesn’t appear to handle chunked http responses from chatGPT, so that abstraction is going to leak too & I'll need to build code around it.


OK, so as we are going to be doing a lot of calls to the server for each render, this is the time for **React Server Components**, yes, let's go, I'm excited! 

### NextJs

OK, to use React Server Components (that look awesome), now I need a **?meta framework?**, ok, **[Next.js](https://nextjs.org/)** looks like the frontrunner.  

:::warning
I'm a little apprehensive about this, looks like this belongs to Vercel, that looks to be an opinionated cloud hosting provider .  I want to deploy to non-opinionated hyper-scale cloud, so concerned the roadmap will be about getting Nextjs to run really well on Vercel or its edge, not my use-case.
:::

Anyway, let's get stuck in!  2 days later, I realise this is no longer a great choice for my app.  I thought I could append server components into the feed,   allowing me to use await on the server to query mongo or chatGPT and returning html to append into the feed, but, I hit 2 big issues:

	* "You cannot import a Server Component into a Client Component" &  "Server Components don’t have lifecycle events (hooks / effects / refs etc)"  This is massive!  No matter how I tried to architect my app, I hit a brick wall. I couldn’t append Server Components to by feed UI  that was a Client Component.  I couldn’t turn the feed into a Server Component because it needed state.
	* I didn’t really want or need a filesystem based router
	

So I made everything a Client component, and using the api folder for server data. It worked, but what value was NextJs now.  I wasn’t happy, this was shambles!  

### <span className="logo small">&lt;<b>/</b>&gt; htm<b>x</b></span>

2 days later, I saw a youtube video on [\</\> htmx](https://htmx.org/). The DOM [swapping](https://htmx.org/docs/#swapping) `beforeend` directive jumped out at me, this looked like all I needed!  But, I could build a webapp without React components, JSX

Turns out I could! And 2 days in, it works, its simple, event using SSE for the chunked chatGPT streaming responses!

I'm sticking with this, lets see how all the other features go, but I have high hopes....

## So, whats next?

My end-2-end tech stack has completely revolved around javascript/typescript for years, I know it very well and I'm fast at getting prototyes out the door.   I need types, they reduce so many bugs/typeos, but I do have a lot of pain with Typescript though, always seem to be fighting with module / import issues etc.

But now, if I no longer need Javascript for client logic, do I stick with it on the server? do I build my server in GO, maybe even .net8, what is the best templating engine?  

My world is turning upside down….

