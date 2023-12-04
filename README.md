<!-- README.md is generated from README.Rmd. Please edit that file -->

# metricminer

`metricminer` is an R package that helps you mine metrics on common places on the web through the power of their APIs.

It also helps format the data so that it can easily be used for a dashboard or other purposes.
It will have an associated [dashboard template](https://github.com/fhdsl/metricminer-dashboard) and tutorials to help you fully use the data you retrieve with `metricminer`  (but these are still under development!)

- You can [read the metricminer package documentation here](https://hutchdatascience.org/metricminer/).
- And you can read more about metric collection in our [associated manuscript -- currently a preprint](https://arxiv.org/abs/2306.03255). 

## Apps supported

Currently `metricminer` supports mining data from:

- [Calendly](https://calendly.com/)
- [GitHub](https://github.com/)
- [Google Analytics](https://analytics.google.com/analytics/academy/course/6)
- [Google Forms](https://www.google.com/forms/about/)
- [Youtube](https://www.youtube.com/)
- [Slido](https://admin.sli.do/events) export files stored on Googledrive

## Data format options

`metricminer`  retrieves API data for you and gives it to you in a format that is a tidy data.frame.
this means metricminer has to be opinionated about what metrics it returns so it fits in a useful data frame easily read by humans.

If you find that the data returned is not what you need you have two options (these options can be pursued concurrently):

1. You can set the `dataformat` argument to `"raw"` to see the original, unedited JSON formatted data as it was returned from the API. Then you can personally look for the data that you want and extract it.
2. You can post a GitHub issue to explain why the metric missing from the data frame of formatted data should be included. And if possible and reasonable, we can work on including that data in the next version of `metricminer`.

## How to install

If you want the development version (not advised) you can install using the `remotes` package to install from GitHub.
``` r
if (!("remotes" %in% installed.packages())) {
  install.packages("remotes")
}
remotes::install_github("fhdsl/metricminer")
```

Attach the library or use decide to use `metricminer::` notation.
```{r setup}
library(metricminer)
```

## Basic Usage

To start, you need to `authorize()` the package to access your data. If you run `authorize()` you will be asked which app you'd like to authorize and whether you'd like to cache that auth information. If you already know which app you'd like to authorize, like `google` for example, you can run `authorize("google")`.

Then follow the instructions on the upcoming screens and select the scopes you feel comfortable sharing (you generally just need read permissions for metricminer to be able to collect data).

```r
authorize()
```

If you want to clear out authorizations and caches stored by `metricminer` you can run:

```
delete_creds()
```

### GitHub

You can retrieve metrics from a repository on GitHub doing this:
```
authorize("github")
metrics <- get_github_metrics(repo = "fhdsl/metricminer")
```

### Calendly

You can retrieve calendly events information using this type of workflow:
```
authorize("calendly")
user <- get_calendly_user()
events <- list_calendly_events(user = user$resource$uri)
```

### Google Analytics

You can retrieve Google Analytics data for websites like this.

First you have to retrieve your account information after you've authorized.

```
authorize("google")
accounts <- get_ga_user()
```

Then you need to retrieve the properties (aka usually the websites you are tracking)
underneath that account.

```
properties_list <- get_ga_properties(account_id = accounts$id[1])
```

Just need to shave off the `properties/` bit from this string.

```
property_id <- gsub("properties/", "", properties_list$properties$name[1])
```

Now we can collect some stats.

In Google Analytics `metrics` are your basic numbers (how many visits to your website, etc.).
```
metrics <- get_ga_stats(property_id, stats_type = "metrics")
```
Whereas `dimensions` are more  a list of events that have happened. So here's a list of people that have logged on.
```
dimensions <- get_ga_stats(property_id, stats_type = "dimensions")
```
Lastly, we have a third option of collecting `link_clicks` and the links they have clicked. This is also known as a dimension according to Google Analytics. However it often isn't compatible for us to download data about link clicks at the same time as other dimension data so in `metricminer` we collect them separately.
```
link_clicks <- get_ga_stats(property_id, stats_type = "link_clicks")
```

### Google Forms

You can retrieve Google form information and responses like this:
```
authorize("google")
form_url <- "https://docs.google.com/forms/d/1Z-lMMdUyubUqIvaSXeDu1tlB7_QpNTzOk3kfzjP2Uuo/edit"
form_info <- get_google_form(form_url)
```

### Slido

If you have used Slido for interactive slide sessions and collected that info and exported it to your googledrive you can use `metricminer` to collect that data as well.

```
drive_id <- "https://drive.google.com/drive/folders/0AJb5Zemj0AAkUk9PVA"
slido_data <- get_slido_files(drive_id)
```
### Youtube 

If you have a channel and the URL is https://www.youtube.com/channel/a_bunch_of_letters_here

Then you can extract stats for the videos on that youtube channel using that URL. 
```
authorize("google")
youtube_stats <- get_get_youtube_stats("a_bunch_of_letters_here")
```

## Bulk Retrievals

Maybe you just want to retrieve it ALL. We have some wrapper functions that will attempt to do this for you.
These functions are a bit more precarious/risky in that there may be reasons certain websites/repos/events/data may not be able to be collected. So collecting repositories one by one will allow you more insight into what is happening.

However, these bulk retrieval functions may help you if you want to grab ALL of your accounts data in one swoop. Just make sure to carefully look over and curate that data after it is attempted to be collected. You may find some retrievals are empty for potentially good reasons (for example if a google form has no responses to collect it will show up with "no responses" in the respective part of the list).

### GitHub bulk

From GitHub you can attempt to collect repository metrics from all repositories from an account.

```
authorize("github")
all_repos_metrics <- get_repos_metrics(owner = "fhdsl")
```

If you want to do this by giving a list of specific repositories you want data from you can just provide a vector of those repository's names like this:
```
repo_names <- c("fhdsl/metricminer", "jhudsl/OTTR_Template")
some_repos_metrics <- get_repos_metrics(repo_names = repo_names)
```

### Google Analytics bulk

Similar to single website retrieval we need to authorize the package.
```
authorize("google")
accounts <- get_ga_user()
```

Then we can provide the account id to `all_ga_metrics` and it will attempt to grab all stats for all website properties underneath the provided account.

```
stats_list <- all_ga_metrics(account_id = accounts$id[5])
```


### Google Forms

As always, we need to authorize the app.
```
authorize("google")
```

We can retrieve a list of form ids using `googledrive` R package.
```
form_list <- googledrive::drive_find(
  shared_drive = googledrive::as_id("0AJb5Zemj0AAkUk9PVA"),
  type = "form")
```

Now we can provide this vector of form ids to `get_multiple_forms`
```
multiple_forms <- get_multiple_forms(form_ids = form_list$id)
```

## Contributions

This is an ever developing package. contact csavonen@fredhutch.org if you are interested in helping us develop `metricminer` Or just file a pull request or issue!
