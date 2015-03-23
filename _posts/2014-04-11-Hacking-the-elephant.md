---
layout: post
category : posts
tags : [php, hacklang, hhvm]
---

On **March 20, 2014** [Julien Verlaguet](https://code.facebook.com/posts/264544830379293/hack-a-new-programming-language-for-hhvm/) announced the official release of [HackLang](http://hacklang.org/) , the new programming language for [HHVM](http://hhvm.com/)


HHVM (aka HipHop Virtual Machine) can run both PHP and Hack code, its goal is to provide an optimized VM for run the existing php application and support the progressive switch to the new Hack features. 

![PHP Elephant]({{ site.url }}/assets/php-elephant.png)

### The language
Hack introduces a lof of new concepts that the PHP still don't knows...
The biggest and maybe the major feature introduced by Hack is the **Static Typing**. Yes now you can develop in a statically typed environment like Java or C# ...


#Cap 1


* Generics
* Constructor Argument Promotion

### Constructor Argument Promotion
Yes this feature is just syntax sugar but in some cases could reduce the code and makes your classes more concise.
Hack automagically defines the class properties according to the constructor's definition (it allows [access modifiers](http://en.wikipedia.org/wiki/Access_modifiers) as well ).
    
{% highlight php %}    
    <?hh
    /**
     * Hack will promote the $name, $description and $price
     * arguments as a class properties.
     */
    class Item
    {
        public function __construct(
                public string $name,
                public string $description,
                public float  $price
            )
        {}
    }
    
    $item = new Item("T-shirt", "Superman T-shirt", 10.90);
    echo sprintf(
            "%s: %s [ %s ]",
            $item->name,
            $item->description,
            $item->price
        );
 {% endhighlight %}

 The code above will produce the following output:
 
 `T-shirt: Superman T-shirt [ 10.90 ]`


### Generics
WTF??
Maybe some PHP developers doesn't have a clue what are generic types, this is absolutely comprensible.
Some modern languages doesn't have generics however these languages like Ruby, Python and PHP support the **[Duck-Typing](http://en.wikipedia.org/wiki/Duck_typing)**.
According to the static typing Hacks introduces Generics support that allows to create generic classes like this...

{% highlight php %}    
    <?hh
    class ItemCollection<T>
    {
        protected Map<string, T> $data;

        public function __construct()
        {
            $this->data = Map<string, T>{ };
        }

        public function add(T $item): Map<string, T>
        {
            $this->data->add(Pair{ spl_object_hash($item), $item });
            return $this;
        }

        public function remove(T $item): Map<string, T>
        {
            $this->data->remove(spl_object_hash($item));
            return $this;
        }
    }
{% endhighlight %}    

In this way we are able to create multiple ItemCollection objects each one with its own item type

`$cart    = new ItemCollection<CartItem>();`

`$library = new ItemCollection<Book>();`

`$zoo     = new ItemCollection<Animal>();`

{% highlight php %}    
    <?hh
    $items = new GenericCollection<Item>();
    $tshirt = new Item(
        "Hawaii",
        "Summer t-shirt"
        10.99
    );
    $codemotion = new ConferenceItem(
        "Codemotion 2014",
        "Codemotion Rome 2014"
        50.00
    );

    $items->add($tshirt);
    $items->add($codemotion);
    $items->remove($tshirt);
{% endhighlight %}    

That and more [new features](http://docs.hhvm.com/manual/en/hacklangref.php) could be introduced gradually into the existing php applications, in fact at this time there is a lot of work behind the main [PHP frameworks](http://hhvm.com/frameworks) to reach the full HHVM compatibility.