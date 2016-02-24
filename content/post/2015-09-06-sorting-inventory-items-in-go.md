---
author: "Adam Presley"
comments: true
date: 2015-09-06T00:00:00Z
tags: ["development", "golang"]
title: "Sorting Inventory Items in Go"
slug: "sorting-inventory-items-in-go"
---

![Sorted Paperwork](/assets/adampresley/images/posts/files-and-clips.png)

Sorting is a fundamental, boring, but useful topic. When developing applications in the business world many times the challenges of sorting are only as complex as an **ORDER BY** clause in a SQL statement. Not all data comes from relational databases, however, and knowing how to perform a basic sort on complex data can be a handy item in the toolbelt. In this post I'll demonstrate how to sort a slice of inventory items. We will even write unit tests so that we enforce good habits right from the start. Let's get started.

<!-- excerpt -->

![Go Sorting Example Diagram](/assets/adampresley/images/posts/go-sorting-example-diagram.png)

The above model diagram illustrates what we'll build in this blog post. Essentially we want to represent an *Inventory* item. Each *Inventory* item is a *Product* with a *ProductSize* and *Price*. Then, when given an array (slice) of inventory items we want to be able to sort them. The criteria for sorting is as follows.

0. Sort first by unique name
0. If two or more product names are the same then sort by size

To follow along with this example you can clone the [repository at Github](https://github.com/adampresley/golang-sorting-example).

Product Size
------------
To start this off let's model the easiest item that has no additional dependencies. A *Product Size* represents the size of a product item. This could be shirt sizes, or the product could have no sizes. With this in mind let's work on a sample set of sizes as follows.

0. Not applicable
0. Extra Small
0. Small
0. Medium
0. Large
0. Extra Large
0. 2-Extra Large

Now we need to define a structure that will best represent the ability to track sizes.

{{< highlight go >}}
package product

type ProductSize struct {
  Abbreviation string
  Size         string
  SortOrder    int
}
{{< / highlight >}}

With this structure we are representing a product size as an abbreviation and a descriptive text. With the **SortOrder** property we are also defining a way to sort the size. This can come in handy for displaying sizes, as well as in sorting. While we are here let's define a set of sizes to make our life easier.

{{< highlight go >}}
var NA ProductSize = ProductSize{"NA", "Not applicable", 1}
var XS ProductSize = ProductSize{"XS", "Extra Small", 2}
var S ProductSize = ProductSize{"S", "Small", 3}
var M ProductSize = ProductSize{"M", "Medium", 4}
var L ProductSize = ProductSize{"L", "Large", 5}
var XL ProductSize = ProductSize{"XL", "Extra Large", 6}
var XXL ProductSize = ProductSize{"XXL", "2-Extra Large", 7}
{{< / highlight >}}

Product
-------
Now that we have a way to define the size of a product, let's define a *Product*. For this example I'll keep this simple. A product is defined by a name and a size. In our example system a *Product* is a thing that is sold, and it only describes the thing.

{{< highlight go >}}
package product

type Product struct {
  Name string
  Size ProductSize
}
{{< / highlight >}}

Inventory
---------
If a *Product* represents the thing we are selling, an *Inventory* item represents how much of a product item you have to sell, and how much it costs. We could also represent information about discounts and warehouse locations, but this example we are only really concerned with the *Product* and *Price*.

{{< highlight go >}}
package inventory

import (
  "fmt"

  "github.com/adampresley/golang-sorting-example/model/product"
)

type Inventory struct {
  product.Product
  Price float64
}

func (inventory Inventory) String() string {
  return fmt.Sprintf("%s - %.2f", inventory.Name, inventory.Price)
}
{{< / highlight >}}

In the above code we are defining a structure for an *Inventory* item. Notice how we use Go's composability features by referencing the *Product* structure. This essentially embeds the product fields into a property named **Product**, but with some added conviences. We also implement the **String()** method to satisfy the [Stringer](https://golang.org/pkg/fmt/#Stringer) interface.

Sorting
-------
Sorting in Go is done via the [sort package](https://golang.org/pkg/sort/). The documentation reveals that to perform a sort of complex object you use the **sort()** method on a type that satisfies the [sort.Interface](https://golang.org/pkg/sort/#Interface) interface. To do this let's create a type that represents a comparator, which will be a collection of *Inventory* items.

{{< highlight go >}}
package inventory

type InventoryComparator []Inventory
{{< / highlight >}}

This by itself is not enough. As the documentation points out we need to implement the **sort.Interface** interface (I know that reads poorly, and I'm sorry). This means our comparator type needs to implement three functions: **Len()**, **Swap()**, and **Less()**. The **Len()** function should return the number of items in the collection. The **Swap()** function should take two integer indexes and swap the two items found at those indexes. And finally the **Less()** function should return true if the item at the first index is less than the item at the second index. If you recall above our first criteria is to sort by the product name. So let's start with that.

{{< highlight go >}}
func (items InventoryComparator) Len() int {
  return len(items)
}

func (items InventoryComparator) Swap(i, j int) {
  items[i], items[j] = items[j], items[i]
}

func (items InventoryComparator) Less(i, j int) bool {
  return items[i].Name < items[j].Name
}
{{< / highlight >}}

Testing
-------
Now let's switch gears a bit. As a matter of good habit, and to ensure our code is working, let's write some tests. For this blog entry I will use [GoConvey](http://goconvey.co/), though you can certainly use the vanilla unit testing framework that comes with Go. Convey just adds a bit of color, and I like the BDD style of writing tests. Let's write a test that ensures that sorting orders inventory items by name.

### First Test
{{< highlight go >}}
package inventory

import (
  "sort"
  "testing"

  "github.com/adampresley/golang-sorting-example/model/product"

  . "github.com/smartystreets/goconvey/convey"
)

func TestInventoryComparator(t *testing.T) {
  /*
   * Let's test the comparator
   */
  Convey("Inventory comparator", t, func() {

    /*
     * Ensure that the comparator sorts by name
     */
    Convey("Sorts by name", func() {
      /*
       * This is our initial inventory array. Notice the names are out of order.
       * When the sort is run this array gets modified directly.
       */
      actual := []Inventory{
        {
          Product: product.Product{Name: "Widget 2", Size: product.NA},
          Price: 59.75,
        },
        {
          Product: product.Product{Name: "Widget 1", Size: product.NA},
          Price: 199.99,
        },
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.L},
          Price: 29.95,
        },
      }

      /*
       * Here is what we expect the result to be.
       */
      expected := []Inventory{
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.L},
          Price: 29.95,
        },
        {
          Product: product.Product{Name: "Widget 1", Size: product.NA},
          Price: 199.99,
        },
        {
          Product: product.Product{Name: "Widget 2", Size: product.NA},
          Price: 59.75,
        },
      }

      /*
       * Sort, then assert that we expect the two arrays to be the same. In
       * GoConvey ShouldResemble does a deep compare.
       */
      sort.Sort(InventoryComparator(actual))
      So(actual, ShouldResemble, expected)
    })
  })
}
{{< / highlight >}}

To start things off we first want to *convey* the idea that we are testing the comparator feature. All comparator tests will then be executed under this function. Our first test attempts to prove that calling sort, using our comparator, will sort the array of inventory items by name. The comparator with the sort method looks like this: ```sort.Sort(InventoryComparator(myUnsortedArray))```.

To do this we start off by creating a slice of *Inventory* items, each with a *Product* and a *Price*. Notice each name is unique and out of order. We are naming this slice **actual** because this is what we will call sort against, and it is the actual result we are testing. Then we build another slice of *Inventory* items called **expected**. This slice has the items in order as we expect them to turn out. The final result is to call sort using our comparator type against **actual**, and we expect that the results should resemble each other. In GoConvey terminology the *ShouldResemble* operator states that it will do a deep comparision of two slices, and although each slice is fundamentally different due to memory locations and allocations, the data should be the same.

Go ahead and run the test, and watch it work! Groovy!

### Second Test
Now let's implement a test to start working on our second sort criteria. When there are two or more inventory items have the same name, but different sizes, we want the sort comparator to order by size. To do this we can use the **SortOrder** column in the *ProductSize* structure to compare two sizes. First, however, let's write the test.

{{< highlight go >}}
    Convey("Sorts by size when the names are the same", func() {
      actual := []Inventory{
        {
          Product: product.Product{Name: "Widget 1", Size: product.NA},
          Price: 59.75,
        },
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.L},
          Price: 29.95,
        },
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.S},
          Price: 29.95,
        },
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.M},
          Price: 29.95,
        },
      }

      expected := []Inventory{
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.S},
          Price: 29.95,
        },
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.M},
          Price: 29.95,
        },
        {
          Product: product.Product{Name: "Dr. Who T-Shirt", Size: product.L},
          Price: 29.95,
        },
        {
          Product: product.Product{Name: "Widget 1", Size: product.NA},
          Price: 59.75,
        },
      }

      sort.Sort(InventoryComparator(actual))
      So(actual, ShouldResemble, expected)
    })
{{< / highlight >}}

If you run the tests now you will see that our new test fails. This is correct and expected. The data now has *Product* items with the same name but differing sizes. Because of this they are not sorted correctly... Yet. Now we want to change the comparator code to make the test pass. Let's see how we can modify the **Less()** function to consider both name and size.

{{< highlight go >}}
func (items InventoryComparator) Less(i, j int) bool {
  if items[i].Name != items[j].Name {
    return items[i].Name < items[j].Name
  } else {
    return items[i].Size.SortOrder < items[j].Size.SortOrder
  }
}
{{< / highlight >}}

The above code is simple. If the names of the products are different then return **true** if the first item is less than the second item (false otherwise). However if the names are the same, then we will return true/false if the size of the product is less than the size of the second product we are comparing to.

![Passing tests](/assets/adampresley/images/posts/golang-sort-passing-tests.png)

Go ahead and run the tests again. You should see a big **PASS** in a box at the top of the screen! Hopefully this is useful to you. Happy coding!