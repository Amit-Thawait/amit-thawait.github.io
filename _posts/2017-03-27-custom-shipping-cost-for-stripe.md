---
title: Custom/Dynamic shipping cost for Stripe Orders
layout: post
published: true
category: programming
tags: [ruby, stripe]
comments: true
---

I am writing this post to basically address two issues that I faced during integrating custom shipping callback with Stripe. Those two issues are <span class="text-red">(Status 402) (Request req_ACwwhHZi9QVD9t) The request timed out while contacting the upstream."</span> and <span class="text-red">Order creation failed while contacting the provider</span>.

I hope you are familier with [Stripe](https://stripe.com/){:target="_blank"} for selling products. If you are selling physical goods then there comes a question of shipping it to the buyer.

Stripe provides 4 options to add shipping cost to the product's total amount which you can use to ship with the help of an external shipping service provider. The 4 different options provided in stripe relay dashboard for shipping are :

1. free: No additional cost, the default.
2. flat_rate: A flat additional cost, regardless of the items ordered, the quantity, or the customer's geographic location. You can even opt to waive the shipping cost above a certain order total.
3. callback: Determined on the fly per order.
4. provider: Calculated using a third party shipping provider like [EasyPost](https://easypost.com/){:target="_blank"} or [Shippo](https://goshippo.com/){:target="_blank"}.

First, lets talk about all the 4 options.

First option : free is self explanatory, means no shipping cost will be charged.

Second option : flat_rate is for charging a flat amount regardless of items ordered, quantity or location which doesn't seems to be a good option because you cannot charge say $20 for the same item to be shipped in US and India.

Third option : callback in which you provide a callback url which returns the shipping option to the order creation call and depending on the options returned by the callback, one of the shipping option can be set on the Stripe Order object and the amount associated with that option will be added to the total bill. This is the most flexible one.

Fourth option : provider wherein you choose one of the shipping providers supported by Stripe : [EasyPost](https://easypost.com/){:target="_blank"} and [Shippo](https://goshippo.com/){:target="_blank"} and with this option the shipping cost is automatically added using the service you have opted for in the stripe dashboard's relay setting which is based on source address, destination address, shipment dimension and weight.

In this blog I will be talking about option 3. I will be explaining some issues that I faced during writing a callback for custom shipping cost.

But before that let me explain why we even went for option 3. First, when we integrated Stripe for payments we used option 4 which was very easy. You just have to update the `order` object created using [Stripe::Order API](https://stripe.com/docs/api#create_order){:target="_blank"} and then depending on the shipping option selected appropriate shipping amount is added to the total amount.

However, there is one issue which we faced with this approach due to the dimension of the product we were shipping. The dimensions of the product were :

width: 10, length: 18, height: 24

weight: 114

width, length, height are in inches and weight is in lbs.

Everything was working fine for 1 or 2 quantity of the item, shipping cost added to the total bill were also proper but we were really surprised to see price jump for quantity = 4. Below is the shipping cost listed for FedEx Ground Delivery for different quantities :

1 : 17.66

2 : 26.05

3 : 56.00

4 : 141.76

5 : 162.48

6 : 

7 :

And as you can see above that the shipping API didn't even returned any shipping amount for quantity 6 and 7.

I knew this issue is from FedEx side but still I contact Easypost support for some guidance related to this issue.

The response I got from one of support staff of Easypost was :

> FedEx ground allows you to ship up to 150 lbs, 108" in length and 165" in length plus girth. So it looks like your 6 item package may be too large to return an available service level.

> Due to the limitations of the carrier, I recommend breaking down the shipment into multiple shipments in order to in order to return a service level.

I then wrote to Stripe support suggesting an improvement : My request was

> I would like suggest an improvement where-in for quantity > 1, shipping cost should be charged as (shipping cost for 1 items * n) for n quantity of products ordered, which can done by creating seperate shipment for each item.

> I am suggesting this because as per my product owner, each hardware box (our product that we ship through FedEx) is shipped as separate shipment as of today. Means, if a person has ordered 3 quantities, it is sent as 3 different shipments via FedEx which costs us "shipping cost for 1 item * 3".

> Since there is a limit for weight and length so, the Easypost/FedEx API will definitely cross that limit even if the product weight and dimension is small and quantities ordered is large.   

> I hope that my suggestion makes sense.

The response I got from Stripe support was :

> We understand what you're saying with regard to larger quantities of small items being shipped in multiple boxes, we are not sure that this is a common enough use case that we'd want to change Relay's shipping calculations.

> Just to clarify, here's the heuristic that Stripe uses for EasyPost shipping calculations:

> 1) Find the maximum dimensions of all items in the order.

> 2) Multiply the maximum height found by the quantity of items in the order

> 3) Add up the weight of the items in the order.

The support guy also suggested me to implement custom shipping option via callback to fit our needs which I also felt would be a better option.

I followed [Stripe's Dynamic Shipping Calculation Article](https://stripe.com/docs/orders/dynamic-shipping-taxes){:target="_blank"} and failed miserably to get it working.

According to this guide you need to create a shipping callback on your server side code and return the response in the format suggested in the above article.

However after reading this article I was unsure about when will my shipping callback action will be called? but still I went ahead and tried to implement it. I implemented the method then tested it and it was returning the response in the appropriate format.

Sample code : 

<script src="https://gist.github.com/Amit-Thawait/c5dcee8c2d7337129f13425222428d8a.js"></script>

<script src="https://gist.github.com/Amit-Thawait/a4763c81c42a8808b8f706f3ebf12cdc.js"></script>

You can refer [Easypost API](https://github.com/EasyPost/easypost-ruby#easypost-ruby-client-library){:target="_blank"} and [Taxjar API](https://github.com/taxjar/taxjar-ruby#taxjar-ruby-gem--){:target="_blank"} to know more about easypost and taxjar api.

Next, I tried integrating this in my website and found out that the request after running for around 20 seconds is getting timeout everytime.

The API call for `Stripe::Order.create` was getting triggered, however the request was never reaching the next line which was `order.selected_shipping_method = 'custom_shipping'`.

I wasn't getting any idea about what am I doing wrong.

I wrote an email to Stripe support regarding this issue :

> After I make Stripe Order create API call, I get an exception as <span class="text-red">(Status 402) (Request req_ACwwhHZi9QVD9t) The request timed out while contacting the upstream."</span>

> The Stripe Order create call itself throws the above exception

> While in the logs after this exception I can see the shipping callback success response.

The response I got from Stripe support was :

>  I've taken another look, and it looks like the problem is that it's taking more than 10 seconds for your endpoint to return a response to Stripe. I'm not sure why the timeout is 10 seconds—I'm still investigating that—but can you tune your code or server to send that response faster? The structure of your JSON looks correct.

I tried this for around 100 times then literally I gave up. Then I got a mail from Stripe support asking if I have got it working or not which was very generous and I thanked them for that.

I also tried this in Rails console and found out that it works through rails console if I execute each command one by one but not via a completed request-response cycle. I was clueless why is this happening.

I wrote about this behaviour to Stripe support and the response I got from Fred (Stripe Support) saved me. His reponse was :

> Just a quick thought here—are you by any chance running this under a single-threaded server such as Webrick? You may not be able to test shipping callbacks in that environment, as the request from Relay to your shipping endpoint will be blocked waiting for the POST request to your server to finish. You might want to look at using a different multi-process backend or pool in order to test this.

That was it actually. I understood that the `shipping_cost` callback is stuck waiting for `Stripe::Order` create call to finish while the `Stripe::Order` create call was waiting for a `shipping_method` attribute to be set on `order` object which gets returned from shipping callback. So, it was a dead-lock case, both waiting for each other to finish which was resulting in Stripe API call timeout issue.

I solved this by running 2 processes of thin server in localhost at 2 different ports and hardcoding the shipping json in callback. Instead of fetching the shipping cost from Easypost I hardcoded it because I wanted to fix my first issue which was related to request timeout. Below are steps on how to do this.

Add `gem thin` to your Gemfile. Run `bundle install` to install thin gem.

You can start 2 process of thin server using the command `thin start --servers 2`

<p align="middle">
	<img src="../assets/images/stripe_custom_shipping/thin_server.png" alt="Thin Server">
	<figcaption align="middle">Thin Server</figcaption>
</p>

Now we need to use second process running on port 3001 for the shipping callback.

We also need to tunneling to localhost which Stripe can use to send the POST request of shipping callback. For this we need to use [ngrok](https://ngrok.com/){:target="_blank"}

Download the installable and go to the installed directory and start ngrok in port 3001. Run

`./ngrok http 3001` in the terminal.

It will start a secure tunnels to your localhost which will look like this :

<p align="middle">
	<img src="../assets/images/stripe_custom_shipping/ngrok.png" alt="ngrok">
	<figcaption align="middle">ngrok tunnel</figcaption>
</p>

Next, you need to copy the https tunnel URL and paste it in stripe dashboard business setting :

<p align="middle">
	<img src="../assets/images/stripe_custom_shipping/stripe_callback_setting.png" alt="callback setting">
	<figcaption align="middle">Stripe shipping callback</figcaption>
</p>

The full callback URL should be something like `https://a06f1d8d.ngrok.io/stripe_payments/shipping_cost` which needs to be entered in callback field shown in the image.
Here, as per rails convention, the above URL points to shipping_cost method of stripe_payments_controller.rb 

After resolving this issue and replacing my hard-coded shipping option with the dynamic one, I was getting request timeout issue again because, my shipping callback was actually taking more than 10 seconds (nearly 12 seconds) because I was making 2 API calls internally, one for easypost and one for taxjar to get the appropriate shipping amount along with tax which you can see in shipping_cost.rb gist I have attached above.

I solved this by firing these two API calls just before Stripe::Order create call, there by preventing timeout of 10 seconds.

Sample order_update json looks like this

<script src="https://gist.github.com/Amit-Thawait/7b624343d2c5a13297049808e62d4126.js"></script>

You need to note that all the amounts mentioned here are in lowest denomination i.e; cents. Hence you need to be careful about passing only integer values in all the amount fields. Passing a float value (which got passed in my case in the amount field of tax_items because of multiplication of a float percent value with the item amount value) will result in an error : <span class="text-red">Order creation failed while contacting the provider</span>.

If you are facing issues related to Stripe custom shipping callback and this post didn't helped in solving one then post a comment and I will try to help you out or else write to stripe support at support@stripe.com explaining your issue and they will definitely get back to you.

I would like thank Fred (from Stripe Support) who helped me to get this working and even following back with me each time I was struck. 