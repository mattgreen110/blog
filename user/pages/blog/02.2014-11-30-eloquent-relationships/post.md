---
title: 'Complex Relationships Using Eloquent'
date: '03-09-2014 00:00'
taxonomy:
    tag:
        - laravel
        - eloquent
        - api
---

If I don't finally start blogging about all things in my life development, [Matt Stauffer](http://mattstauffer.co/) is going to kill me. So this is a first of hopefully many posts (because I don't want to die). Of murder... from Matt Stauffer.

The primary purpose of this post is to prove that when you need to work with complex relationships that are deeply nested and require many options using Eloquent, there are many things you can do to make your life easier and minimize the pain-in-the-assness that will ensue unless you carefully plan your architecture.

#####The Struggle Is Real!
If you are like me, when you start using Eloquent (or any tool for that matter), you jump right in. I read the docs and immediately start trying things. Documentation shows very simple examples and if your application is simple enough it just WORKS and you save a lot of time. However, as things get more complex you start running out of examples in the documentation you can replicate or that directly apply to your use case. Maybe then you find answers at stack exchange or someone else who figured out how to do what you are doing but it's inevitable that at some point you are just have to figure it out on your own and the spoon feeding is over.

Consider Eloquent. The documentation shows you how to call a model with its relationships and even how to order that relationship if you use a callback function. But what if you want to order your parent model by one of its child fields? For example, let's say you have a product model that has a category relationship and you want to order the products by the category name and display them in a table in your view. The docs don't spell this out for you. It's possible to do but you won't find a clear example.  

#####A Practical Example
I think some context is helpful at this point. I have a custom CMS (Laravel) that manages product data and outputs an API that feeds that data to other systems within the company for which I work. Different clients consuming it include a public-facing website (ExpressionEngine), a portal for the salesforce and distributors (Craft CMS), an internal intranet (EE) and a mobile app (Ionic/Angular/Cordova). These clients (which are just websites using a CMS) use an API wrapper that I wrote to make API calls and communicate back and forth to the API. Just remember that the API output involves a bunch of nested relationships and that is why this is important.

The products model has relationships that include locations, assets, categories and divisions. Some of these relationships also have their own relationships like sections and category groups. Consider that the API allows for pagination, sorting and ordering by data fields. It also needs to allow for ordering a model by its relationship fields and specifying parameters on those relationships. The logic and calls can get out of hand fast especially if you leave any logic up to the client view (bad). This led to one major goal:

**Minimize API calls from any client, keeping it at ONE as much as possible for each client view.**

You want to leave all of the business logic up to the API. This works much better for performance and gives you a clean view template. This means I don't want to call a categories model and then scoop up the products for that category which are now filtered for a specific category. If I want products, I want to call the products model. If I need to specify a category, I need to tell my API that and then let it work that logic out on its own. The less the clients need to know the better.

Let's start with a really horrible example to illustrate a point. This would happen in my view that displays a product along with all of its categories and its related assets. This is using my API wrapper:

<pre><code class="language-php">

    // Data model that contains first 20 products
    $products = $this->api->products->page(1)->limit(20)->get();

    // Display my products in a view
    foreach ($products as $product)
    {
        // Gross if statement that proves I am calling stuff I'll never use
        if ($product->location == 'US' && $product->category == 'category')
        {
            // Do a bunch of viewey stuff to show product data.
            echo "<h2> $product->name </h2>";
            echo "<p> $product->description </p>";

            // Oh but now we need the product assets
            // Let's make another call for those
            $assets = $this->api->assets->get();

            // Another loop to display assets for this product
            foreach ($asset as $asset)
            {
                // Another gross if statement because we need
                // to make sure we only show assets for this product
                if ($asset->product_id == $product->id)
                {
                    // More viewey stuff for assets
                    // Oh crap we need those asset's categories so let's do
                    // ANOTHER loop and make even more calls
                    $asset_categories = $this->api->categories;

                    foreach ($asset_categories as $categories)
                    {
                        // Go home view logic you are drunk
                    }
                }
            }
        }
    }

</code></pre>

Okay okay I know it's an extreme example of badness but still. It kills performance by making all of these calls within these loops. This is potentially hundreds of queries and API calls and it's retrieving data you aren't even using since it's filtered through if statements in your view. That's no good. It might not be this bad, but you may still want to make other calls within loops for convenience, or call different models for filtering which might confuse you in 6 months.

Instead I want to make ONE call and use Eloquent's eager loading. The result BEFORE caching is actually just **5 total queries!** So that would look more like this using my wrapper again:

<pre><code class="language-php">

    // Data model - I am filtering when making the call,
    // not calling everything and using if statements
    $products = $this->api->products->category('category')
                                    ->location('US')
                                    ->display('public')
                                    ->orderby('name')
                                    ->sort('asc')
                                    ->page(1)
                                    ->limit(20)
                                    ->get();

    foreach ($products as $product)
    {
        // Need Categories? Already got them
        foreach($product->categories as $categories)
        {
            // Do something with product categories
        }

        // Need Assets? Already got them
        foreach($product->assets as $assets)
        {
            // Do something with product assets

            // Need Asset Categories? Already got them
            foreach($asset->categories as $category)
            {
                // Do something with asset categories
            }
        }
    }
</code></pre>

**One call, no logic in the view** <br>
I tell the API everything I need and ONLY what I need in the API call and all relationships are nested within the output. Will this work for every situation? No. But I have everything I need for this view. Seeing how loading an empty template in EE alone means you have already made double digit queries, we need all of the help we can get. EE makes more requests than my 5-year-old daughter if we were in a store full of Disney's "Frozen" merchandise.

#####So What Next?

**How might this look in Laravel using Eloquent? How do you tell Eloquent "hey give me this data and its relationships all at once based on X and Y?"**

I will write a follow-up post showing examples of the architecture I am using for that, but the groundwork for making this happen or rather even allowing for this to be possible includes:

* Keep all business logic separate from persistence.
* De-coupling from the framework (Laravel/Eloquent) as much as possible.
* Use a repository to decorate my model with filter params from input.

