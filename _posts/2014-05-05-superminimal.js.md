---
title: superminimal.js [ JS __proto__ ]
layout: post
category : posts
tags : [javascript, prototype]
---

One of the main difference between **javascript** and other languages is the *prototypal inheritance* that make possible to inherit properties and behaviour through object prototyping.


Having no classes the prototyping is a very powerful feature that allows one object to inherit the properties of another.
If we use it well, this can reduce object initialization time and memory consumption.


>JavaScript has a class-free object system in which objects inherit properties directly from other objects. This is really powerful, but it is unfamiliar to classically trained programmers.
                         -  (Douglas Crockford) Javascripts the good parts -


Thi mechanism allow us to extend the objects features in a few line of code.
A good example of this is the following code that create a render method for javascript strings.

{% highlight javascript %}
    String.prototype.render = function(data){   
        return this.replace(
            /\{([^{}]*)\}/g,
            function(match, group){
                var value = data[group];
                if( typeof value === 'function' ) {
                    return value.apply(data);
                } else if(
                    typeof value === 'number' || typeof value === 'string'
                    ) {
                    return value;
                }

                return match;
            }
        );
    };
{% endhighlight %}

**WOW!!** we've create a *superminimal* javascript templating system.


A simple use case could be
{% highlight javascript %}
    var msg = "Hi {name}, welcome to the {brand} platform.".render({
        name: "John",
        platform: "Github"
    });

    console.log(msg); 
    /* Hi John, welcome to the Github platform. */
{% endhighlight %}

... but a more complex case is trivial as well

{% highlight javascript %}
    var order = {
        user: {
            name: "John",
            last_name: "Doe"
        },
        delivery: {
            address: "Boulevard avenue, 345",
            state:   "California",
            date:    "2014-01-04"
        },
        code: "GH670LOI-90"
    };

    var msg = "Hi {name} your order: {code} will be shipped to {address} at {date}".render({
        order: order,
        name: function(){
            return "{first_name} {last_name}".render({
                first_name: this.order.user.name,
                last_name: this.order.user.last_name
            })
        },
        code: order.code,
        address: function () {
            return "{address} [{state}]".render({
                address: this.order.delivery.address,
                state:   this.order.delivery.state
            })
        },
        date: order.delivery.date
    });

    console.log(msg);
    /* Hi John Doe your order: GH670LOI-90 will be shipped to Boulevard avenue, 345 [California] at 2014-01-04 */
{% endhighlight %}

In ten line of code we've realized a very useful templating sysyem.
Try to use is within the browser 
{% highlight html %}
    <html>
    <head>
        <title>Try String.render({...})</title>
    </head>
    <body>

    <script src="https://gist.githubusercontent.com/toretto460/0b139fdc746a88161a49/raw/2662b31751bc331e8b05fae46bb642d0df646b8b/superminimal.js"></script>
    <script type="text/javascript">
        alert("Welcome to the {site_title}".render({site_title: 'Try String.render({...})'}));
    </script>
    </body>
    </html>
{% endhighlight %}