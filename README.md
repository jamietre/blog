# blog

Testing new github pages

Here's some code

```Javascript
// money = [{ category, amount }, ... ]
// group all sale transactions by category, and return the
// # and total $ of sales per category

let money = iter(allSales)
    .groupBy('category')
    .map(([category, sales])=> {
        return {
            volume: sales.length,
            totalDollars: iter(sales).map(e=>e.amount).sum()    
        }
    })
```

And now:

* Some
* Bullets

### subheading

The end!
