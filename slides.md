## Title
=> Title
## Special Shoutouts
- Brazil literally the GOAT, these two Gentlemen enabled a beautiful Dev Ops experience that made all of this as possible as it was and I cannot thank them enough. 
	- Wagner Silva
	- Gilmar Lima
- Also need to give credit where credit is due for not saying no when I asked about writing the applications in rust and for being fantastic leads
	- Steven Leve
	- Todd Weaver
## Source Code
- We only have our code in GitLab which requires account and set up on our end, happy to provide read only access to our Git Lab for anyone who might be interested in looking at the Source code 
## What this talk is not
- Not a talk about architecture
- While this *is* a talk about Rust, it is *not* a talk about learning Rust, just the benefits of *why* you should think about learning Rust
## Who's is this guy? 
- Good Question! 
	- I'm Nico, Senior Software Engineer on America's Engineering team
	- Most of my work revolves around the backend of ShareASale sale processing and as you'll hear in the presentation, the last couple rounds of projects have all primarily been in Rust
- Started learning Rust and Go about 2 ish years ago for my personal side projects and  very quickly dropped Go for full speed ahead with Rust
## What is Rust?
- Memory safe high performant language with no runtime or garbage collector
- Cross platform compatible (eg. build on linux for Windows)
- Strict type system and ownership model that garuntees memory and thread safety
## History Lesson of the ShareASale Platform
### The ancient texts were written
- Frontend and Backend Started as just Cold Fusion 
- some services and jobs were ported from ColdFusion to C# due to higher performance of C#,
- Later on we introduced our shopify plugin + pixel with Node/Js and our woo commerce plugin which is PHP
- Also Introduced some Python surrounding our data feed application
### "Forced" Adoption of Containerization
#### A messy clutter of Tech debt and sorrow (aka: why windows server EC2 instances are not the future)
- As great as some of our sale processing pipeline is, there's also a few really bad pain points on the database which luckily for us means if we can think cloud native and remove a dependency on those already slightly constrained resources
- This causes us to gain flexibility AND speed from not having to rely on legacy architecture and on Windows server and move into the brilliant world of Linux containers
- And since Rust is platform agnostic it can Compile on windows, Linux, or Mac depending on your preferred tool/OS stack. 
- With the planned sunset of the ShareASale platform as we move through the upgrade, we gained a huge opportunity of making some of these one off services to solve a need since we won't need it for much longer.  
## Introduction of Rust
#### Project 1:  Analytics API
- Problem to solve: Shopify pixel was failing to finish the checkout event and we had no way to log those errors as they were happening on the client side and not on the server side
- Logging endpoint used for logging client side errors and Shopify checkout pixel events and stores them in a local sqlite file that is exported to an AWS S3 bucket 
- This was used to cover the logging gap for debugging and resolving errors occurring within the shopify checkout event firing as those events are handled on the client's browser 
#### Project 2:   Transaction Sync Service
- Problem to solve: After a merchant/Advertiser is upgraded to an Awin account but the click was registered on the ShareASale account, we needed a way to take the transaction that would have been created on SAS and send it to CTS to convert with all the proper details validated (click is valid and hasn't expired, Amount > 0, valid merchant balance, and so on)
- Reads from 2 different AWS SQS and converts what would normally be a ShareASale transaction and sends it to CTS to convert it to an Awin Transaction 
- Special thanks and acknowledgments to Pradeep Hunjanalu and Andrea Fioroni for also helping contribute towards the project :)
#### Project 3: CloudflareWorker KeyValue store uploader
- Problem: During the early days of tracking migration, we needed a way to split the traffic and clone a single sale and click to both SAS and Awin sale processing systems as well as control how a given merchant ID should be handled (ie larger clients should NOT be sent over and should be minimally impacted in case we did something wrong or had an unoptimized code logic causing a slowdown during peak hours)
- Reads from input csv file and either adds to or replaces a merchant ID to the cloudflare Key/value store.
- That queue was how we enabled an allow list for testing the first stages of the transaction sync and tracking migration journey to mirror the ShareASale traffic over to CTS
#### Project 4: Tiger Claw
- Problem: The Tiger team was using a mix of curl and postman to manually test various migration API endpoints and selfishly I did not like manually clicking around. A slightly side problem was that I had been doing mainly CTS/Tracking migration and came back late to the Tiger team so I had little visibility and ideas on how the Migration API architecture behaved
- personal CLI testing tool used to retrieve auth key and make requests to the Team Tiger Migration API 
- replaced manual curl or postman for testing the Growth Account Migration service API or SaS Data Import Service. 
## Cons
- Borrow checker is not a terribly complicated error to solve but was one of the more nuanced hurdle to get over
	- It eventually gets a lot easier but a substantial friction to learn to deal with is around identifying the actual cause of your borrow rather than where the borrow errors. 
	- I've found this issue usually occurs from passing a struct over to a method without declaring that as a reference or I've forgotten to pass the value of a mutable variable as a reference. 
- It's a lot harder to write than javascript which means developers without any previous experience in Rust or a similar pointer heavy language like C, C++, or Zig etc. 
	- Fair point, but also worth noting that the power of a pointer based language will almost certainly beat out javascript every day of the week (assuming it's built well)
	- but it's also a lot harder to write good manual memory management like in C or working with the much stricter Rust compiler 
	- Although not that difficult of a concept, Rust's handling of memory safe makes it harder to write bad code can occasionally throw you for a bit of a loop in figuring out what how to resolve those kinds of errors.
- Build times with AWS related packages can get real lengthy if you're not careful. 
	- Still working on our build pipeline to cache already compiled dependencies and enable incremental build to re-use old build artifacts that haven't been updated since the last time the service was built
## Pros
- Compiler not only tells you what is wrong but in a large majority in cases tells you exactly how to solve it
	- When you add tools like `Clippy` on top of the compiler you get easily enforceable rules with system, user, repo, or file level error and warning rules about what coding standards you want to enforce
	- When mistyping a variable it will also error and tell you which variable it thinks you're trying to use which is beautiful
- out of the box rust is cross OS compatible 
	- Support almost anywhere between the server, client, desktop, mobile, and even for the Nintendo gameboy [https://github.com/bushrat011899/bevy_mod_gba](https://github.com/bushrat011899/bevy_mod_gba)
	- You can cross compile from one OS platform to another
- tooling is great
	- clippy
	- bacon
	- cargo
	- rust-analyzer
	- rustup
	- rustdoc
	- RustRover
	- Rustaceanvim, neovim plugin
- NASA likes it and NASA does cool space stuff
	- There is no more information I just thought this was cool
## Frictions and pitfalls
- AI usually is good at knowing how to use the std library but can lead to really unoptimized code with adding `.clone()`  instead of borrowing the reference and other such rust anti patterns
- I had the luxury of coming into learning with about a year-ish of C and C++ which helped a lot with understanding the intricacies of ownership/borrowing but even with that knowledge and experience the nuances of rust run deep
- Even though I enjoyed Leptos (rust web framework that behaves similar to React and other other similar component based frameworks) I cannot in good faith say that it has as much ease as javascript and all of it's various frameworks
## Conclusion 
- Do I think it's the ultimate tool to solve every problem? no
- Do I think it's an amazing tool when fast and high risk type safety is concerned? Yes
- Do I think we should introduce it to our Awin Tech Radar? yes
##  QA
- Floor is yours if there are any questions
##  Links
- https://www.rust-lang.org/
