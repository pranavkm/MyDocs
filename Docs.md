* things to talk about
    - goal of data collection
        - Ask the audience - "Serverside blazor has app state, component state for the duration of a circuit on the server. Point to SpicyBoi - what's your reaction when I tell you this?"
        - Customers have the same concerns you do. It's our job to sell them on the viability of this model. We have to collect data, to convince them that this is viable model.
        - Blazor is new concept. It's SignalR + WPF (?). We understand the scalability and performance characteristics of various parts of the system, but not the whole.
        - Set out to investigate this knowing that we're starting late, and starting from scratch with limited time to react prior to 3.0.
            - What are the characterstics of running a Blazor app
            - What does it look like to run a normal sized in a normal sized VM? Also determine "normal"?
            - How do you define a well-performing Blazor app?

    - techniques used
        - Ignitor. No Selenium.
            - What is Ignitor? Etymology of the term.
            - Wanted something lightweight and scalable. Selenium is not either.
            - We wanted to test things that well behaving clients would not do.
        - What ignitor code looks like:
            - You trigger an event, wait for DOM update, and then check for an Id.
            - This is fairly similiar with what you would do with Selenium.
            - You write "test" code that is tied to the server. But this is ok

    - experiments and findings
        - This is work in progress. We got some preliminary data and plan to continue iterating.
        - Scenarios: what does it look like


        -
    - what were the findings?


-----------------------------------------------------------------------

Production-ready server-side Blazor

ASP.NET Core 3.0 ships with the first major release of server-side Blazor. Server-side Blazor builds on top of ASP.NET Core & SignalR. We understand the performance characteristics, programming patterns, and ways to benchmark these parts. Server-side Blazor introduces a fair bit of novelty to the mix. In this article, we'll look at how we went about profiling, our guidance for capacity planning, as well some DOs and DONTs when authoring a server-side application.

### Benchmarking the framework

The Blazor framework has an in-memory representation of the application's UI state. Events - either a browser UI event such as clicking a button or navigating, or internally raised by the application such as a timer, may cause Components to change this UI state. Blazor reconciles the previous and current UI state and produces a patch, or a diff. A single UI event may produce multiple diffs. Diffs are translated as operations to the HTML DOM. When applied in order, the UI visible in the browser syncs with the server's representation of this state. In WebAssembly-based Blazor, DOM updates are sent over WebAsssembly-JavaScript interop. In server-side Blazor, updates are sent to the client over a SignalR connection. In both of this cases, this UI-rendering loop is at the heart of Blazor.

We realized that testing the framework required a client that understood the diffing protocol, knew how to raise DOM events, and when the DOM has updated. Using a browser-automation like Selenium seemed the obvious first choice. However, it has some limitations: (a) we wanted to automate a really large number of clients. Browser automation really doesn't scale well at the level of 1000s of instances. (b) we wanted to explore the effects of a non-conforming client on the server. Both of these requirements led us to writing a Blazor-specific client named *Ignitor*. We intend on making it available to developers in a future release. A typical usage for Ignitor looks like this:

```C#
// Find the item tracking the count of pizzas added
var pizzaOrders = client.ElementHive.FindElementById("pizzaOrders");
var pizzaCount = pizzaOrders.GetAttribute<int>("pizzaCount");

// Wait for the 'cheese' pizza option to be visible and click on it
var pizza = await client.WaitForElementAsync("cheese", cancellationToken);
await pizza.ClickAsync(client.HubConnection, cancellationToken);

// Click on the 'Confirm' option once the dialog pops up
var confirmButton = await client.WaitForElementAsync("Confirm", cancellationToken);
await confirmButton.ClickAsync(client.HubConnection, cancellationToken);

// Wait for the count of pizzas to update
await client.WaitUntil(_ => pizzaOrders.GetAttribute<int>("pizzaCount") > pizzaCount);
```








