# google-play-scraper

Golang scraper to get data from Google Play Store

This project is inspired by the [google-play-scraper](https://github.com/facundoolano/google-play-scraper) node.js project

## Instalation

```shell
go get -u github.com/n0madic/google-play-scraper/...
```

## Usage

### Get app details

Retrieves the full detail of an application.

```go
package main

import (
    "github.com/davecgh/go-spew/spew"
    "github.com/n0madic/google-play-scraper/pkg/app"
)

func main() {
    a := app.New("com.google.android.googlequicksearchbox", app.Options{
        Country:  "us",
        Language: "us",
    })
    err := a.LoadDetails()
    if err != nil {
        panic(err)
    }
    err = a.LoadPermissions()
    if err != nil {
        panic(err)
    }
    spew.Dump(a)
}
```

### Search apps

Retrieves a list of apps that results of searching by the given term.

```go
package main

import (
    "fmt"

    "github.com/n0madic/google-play-scraper/pkg/search"
)

func main() {
    query := search.NewQuery("game", search.PricePaid,
        search.Options{
            Country:  "ru",
            Language: "us",
            Number:   100,
            Discount: true,
            PriceMax: 100,
            ScoreMin: 4,
        })

    err := query.Run()
    if err != nil {
        panic(err)
    }

    errors := query.LoadMoreDetails(20)
    if len(errors) > 0 {
        panic(errors[0])
    }

    for _, app := range query.Results {
        if !app.IAPOffers {
            fmt.Println(app.Title, app.URL)
        }
    }
}
```

### Get category

Returns a list of clusters for the specified application category.

```go
package main

import (
    "fmt"

    "github.com/n0madic/google-play-scraper/pkg/category"
    "github.com/n0madic/google-play-scraper/pkg/store"
)

func main() {
    clusters, err := category.New(store.Game, store.SortNewest, store.AgeFiveUnder, category.Options{
        Language: "us",
        Number:   100,
    })
    if err != nil {
        panic(err)
    }

    err = clusters["Top New Free Games"].Run()
    if err != nil {
        panic(err)
    }

    for _, app := range clusters["Top New Free Games"].Results {
        fmt.Println(app.Title, app.URL)
    }
}
```

### Get collection

Retrieve a list of applications from one of the collections at Google Play.

```go
package main

import (
    "fmt"

    "github.com/n0madic/google-play-scraper/pkg/collection"
    "github.com/n0madic/google-play-scraper/pkg/store"
)

func main() {
    c := collection.New(store.TopNewPaid, collection.Options{
        Country: "uk",
        Number:  100,
    })
    err := c.Run()
    if err != nil {
        panic(err)
    }

    for _, app := range c.Results {
        fmt.Println(app.Title, app.Price, app.URL)
    }
}
```

### Get developer applications

Returns the list of applications by the given developer name or ID

```go
package main

import (
    "fmt"

    "github.com/n0madic/google-play-scraper/pkg/developer"
)

func main() {
    dev := developer.New("Google LLC", developer.Options{
        Number: 100,
    })
    err := dev.Run()
    if err != nil {
        panic(err)
    }

    for _, app := range dev.Results {
        fmt.Println(app.Title, "by", app.Developer, app.URL)
    }
}
```

### Get similar

Returns a list of similar apps to the one specified.

```go
package main

import (
    "fmt"

    "github.com/n0madic/google-play-scraper/pkg/similar"
)

func main() {
    sim := similar.New("com.android.chrome", similar.Options{
        Number: 100,
    })
    err := sim.Run()
    if err != nil {
        panic(err)
    }

    for _, app := range sim.Results {
        fmt.Println(app.Title, app.URL)
    }
}
```
