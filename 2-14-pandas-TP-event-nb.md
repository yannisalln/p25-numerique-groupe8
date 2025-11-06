---
jupytext:
  custom_cell_magics: kql
  text_representation:
    extension: .md
    format_name: myst
    format_version: 0.13
    jupytext_version: 1.17.3
kernelspec:
  display_name: Python 3 (ipykernel)
  language: python
  name: python3
---

# grouping by period and category

+++

before starting this TP:
- put the `2-14-pandas-TP-event-nb.md` file in your github repository `p25-numerique-groupe8`  
- put the file `events.csv` and the file `country.csv` in its `data` folder 
- put the files `result-color-w.png`, `result-color-m.png`, `result-color-y.png`,  
   `result-bw-w.png`, `result-bw-m.png` and `result-bw-y.png` in its `media` folder

+++

in this TP we work on 

- data that represents *periods* and not just one timestamp
- checking for overlaps
- grouping by period (week, month, year..)
- then later on, grouping by period *and* category
- and some simple visualization tools

+++

here's an example of the outputs we will obtain

````{grid} 3 3 3 3
```{image} media/result-color-w.png
```
```{image} media/result-color-m.png
```
```{image} media/result-color-y.png
```
````

+++

## imports

```{code-cell} ipython3
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
```

## the data

we have a table of events, each with a begin (`beg`) and `end` time; in addition each is attached to a `country`  
(we do not yet know what these events are)

```{code-cell} ipython3
events = pd.read_csv("data/events.csv")
events.head(10)
```

### adapt the type of each columns

surely the columns dtypes need some care
hints:
1. use the the `datetime` formats described here https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior
2. or find and use the international format described https://en.wikipedia.org/wiki/ISO_8601

```{code-cell} ipython3
events['beg'] = pd.to_datetime(events['beg'])
events['end'] = pd.to_datetime(events['end'])
```

```{code-cell} ipython3
# check it

events.dtypes
```

### raincheck

check that the data is well-formed, i.e. **the `end`** timestamp **happens after `beg`**

```{code-cell} ipython3
df_beg_end = events['beg'] >= events['end'] # df où True si beg < end
df_beg_end.any() # renvoie False donc y'a pas de True donc tout est bon
```

sort the dataframe by the `'beg'` column

+++

### are there any overlapping events ?

+++

check if there are overlapping events  

hints:
1. you can use the `pandas.Series.shift` method  
  (i.e. the method `shift` applied to a `pandas` `Series`)
2. there is a `pandas.Timedelta` function

```{code-cell} ipython3
# pd.Series.shift?
```

```{code-cell} ipython3
events_beg_sort = events.sort_values('beg') # on trie le tableau pour pouvoir plus simplement comparé les intervalles
events_end_shift = events_beg_sort['end'].shift(1)

events_beg_sort = events_beg_sort.iloc[1:] # on supprime la première ligne car on a décalé les index de 1
events_end_shift = events_end_shift.iloc[1:] # idem

(events_beg_sort['beg'] < events_end_shift).any()
# la réponse est False donc aucun chevauchement !
```

### timespan

What is the timespan covered by the dataset (**earliest** and **latest** events, and **duration** in-between) ?

```{code-cell} ipython3
print(events_beg_sort.iloc[0])
```

```{code-cell} ipython3
print(events_beg_sort.iloc[-1])
```

```{code-cell} ipython3
total_time = events_beg_sort['end'].iloc[-1] - events_beg_sort['beg'].iloc[0]
print(total_time)
```

### aggregated duration

so, given that there is no overlap, we can assume this corresponds to "reservations" attached to a unique resource  
write a code that computes the **overall reservation time**, as well as the **average usage ratio** over the overall timespan

keep a column with the duration of each event

```{code-cell} ipython3
events_beg_sort['duration'] = events_beg_sort['end'] - events_beg_sort['beg']
total_duration = events_beg_sort['duration'].sum()
print(total_duration)

ratio = total_duration / total_time
print(f"{ratio*100:.2f} %")
```

## visualization - grouping by period

### usage by period

grouping by periods: by week, by month or by year, plot the `bar` of the **total duration in that period**

There are at least 2 options to do this grouping, based on `resample()` and `to_period()` : **write them both**

(you can access the `dt` (datetime) attibuts and methods of a column if needed)


`````{admonition} for now, **just get the grouping right (do not improve your plots yet)**
:class: dropdown

you should produce something like e.g.

````{grid} 3 3 3 3
```{image} media/result-bw-w.png
```
```{image} media/result-bw-m.png
```
```{image} media/result-bw-y.png
```
````
we'll make cosmetic improvements below, and [the final results look like this](#label-events-output), but let's not get ahead of ourselves
`````

```{code-cell} ipython3
# pd.DataFrame.resample?
```

```{code-cell} ipython3
events_beg_sort_index = events_beg_sort.set_index('beg')
```

```{code-cell} ipython3
events_gb_week_resamp = events_beg_sort_index['duration'].resample('W').sum()
events_gb_month_resamp = events_beg_sort_index['duration'].resample('M').sum()
events_gb_year_resamp = events_beg_sort_index['duration'].resample('Y').sum()
```

```{code-cell} ipython3
# events_gb_week_resamp.plot.bar()
# plt.show()
# events_gb_month_resamp.plot.bar()
# plt.show()
# events_gb_year_resamp.plot.bar()
```

```{code-cell} ipython3
# pd.Series.dt.to_period?
```

```{code-cell} ipython3
events_beg_sort['week'] = events_beg_sort['beg'].dt.to_period('W')
events_gb_week_toperiod = events_beg_sort.groupby('week')['duration'].sum()

events_beg_sort['month'] = events_beg_sort['beg'].dt.to_period('M')
events_gb_month_toperiod = events_beg_sort.groupby('month')['duration'].sum()

events_beg_sort['year'] = events_beg_sort['beg'].dt.to_period('Y')
events_gb_year_toperiod = events_beg_sort.groupby('year')['duration'].sum()
```

```{code-cell} ipython3
# events_gb_week_toperiod.plot.bar()
# plt.show()
# events_gb_month_toperiod.plot.bar()
# plt.show()
# events_gb_year_toperiod.plot.bar()
```

### improve the title and bottom ticks

add a title to your visualisations

also, and particularly relevant in the case of the per-week visu, we don't get to read **the labels on the horizontal axis**, because there are **too many of them**  
to improve this, you can use matplotlib's `set_xticks()` function; you can either figure out by yourself, or read the few tips below

````{admonition} a few tips
:class: dropdown tip

- the object that receives the `set_xticks()` method is an instance of `Axes` (one X&Y axes system),  
  which is not the figure itself (a figure may contain several Axes)  
  ask google or chatgpt to find the way you can spot the `Axes` instance in your figure
- it is not that clear in the docs, but all you need to do is to pass `set_xticks` a list of *indices* (integers)  
  i.e. if you have, say, a hundred bars, you could pass `[0, 10, 20, ..., 100]` and you will end up with one tick every 10 bars.
- there are also means to use smaller fonts, which may help see more relevant info
````

```{code-cell} ipython3
# let's say as arule of thumb
LEGEND = {
    'W': "week",
    'M': "month",
    'Y': "year",
}

SPACES = [
    'W': 12,   # in the per-week visu, show one tick every 12 - so about one every 3 months
    'M': 3,    # one every 3 months
    'Y': 1,    # on all years
]
```

```{code-cell} ipython3
fig, axes = plt.subplots(1, 3)

events_gb_week_toperiod.plot.bar(ax=axes[0])
axes[0].set_xticks(range(0, len(events_gb_week_toperiod), 12)) # on ne prend qu'une semaine sur 12

events_gb_month_toperiod.plot.bar(ax=axes[1])
axes[1].set_xticks(range(0, len(events_gb_month_toperiod), 3)) # idem mais tous les 3 mois

events_gb_year_toperiod.plot.bar(ax=axes[2])
plt.show()
```

### a function to convert to hours

you are to write a function that converts a `pd.Timedelta` into a number of hours  
1. read and understand the test code for the details of what is expected
2. use it to test your own implementation

note that if an hour has started even by one second, **it is counted**

```{code-cell} ipython3
# the type of timedelta is pd.Timedelta
# the function returns an int
def convert_timedelta_to_hours(timedelta: pd.Timedelta) -> int:
    total_sec = timedelta.total_seconds()
    one_hour = 3600
    return int(np.ceil(total_sec/one_hour))
```

```{code-cell} ipython3
# test it

# if an hour has started even by one second, it is counted
test_cases = ( 
    # input in seconds, expected result in hours
    (0, 0), 
    (1, 1),     (3599, 1),     (3600, 1), 
    (3601, 2),  (7199, 2),     (7200, 2), 
    # 2 hours + 1s -> 3 hours
    (7201, 3),  
    # 3 hours + 2 minutes -> 4 hours
    (pd.Timedelta(3, 'h') + pd.Timedelta(2, 'm'), 4),
    # 2 days -> 48 hours
    (pd.Timedelta(2, 'D'), 48),
)

def test_convert_timedelta_to_hours():
    for seconds, exp in test_cases:
        # convert into pd.Timedelta if not already one
        if not isinstance(seconds, pd.Timedelta):
            timedelta = pd.Timedelta(seconds=seconds)
        else:
            timedelta = seconds
        # compute and compare
        got = convert_timedelta_to_hours(timedelta)
        print(f"with {timedelta=} we get {got} and expected {exp} -> {got == exp}")

test_convert_timedelta_to_hours()
```

```{code-cell} ipython3
# for debugging; this should return 48

convert_timedelta_to_hours(pd.Timedelta(2, 'D'))
```

### use it to display totals in hours

keep the same visu, but display **the Y axis in hours**  
btw, what was the unit in the graphs above ?

hint:  
you can use `map` to apply a function (for example `convert_timedelta_to_hours`) to a `pandas.Series`

```{code-cell} ipython3
# l'unité précédente était des ns
events_gb_week_toperiod.dtype
```

```{code-cell} ipython3
events_gb_week_toperiod_hours = events_gb_week_toperiod.map(convert_timedelta_to_hours)
events_gb_month_toperiod_hours = events_gb_month_toperiod.map(convert_timedelta_to_hours)
events_gb_year_toperiod_hours = events_gb_year_toperiod.map(convert_timedelta_to_hours)
```

```{code-cell} ipython3
fig, axes = plt.subplots(1, 3)

events_gb_week_toperiod_hours.plot.bar(ax=axes[0])
axes[0].set_xticks(range(0, len(events_gb_week_toperiod), 12)) # on ne prend qu'une semaine sur 12

events_gb_month_toperiod_hours.plot.bar(ax=axes[1])
axes[1].set_xticks(range(0, len(events_gb_month_toperiod), 3)) # idem mais tous les 3 mois

events_gb_year_toperiod_hours.plot.bar(ax=axes[2])
plt.show()
```

## grouping by period and region
the following table allows you to map each country into a region

```{code-cell} ipython3
countries = pd.read_csv("data/countries.csv")
countries.head(3)
```

### a glimpse on regions

what's the most effective way to see how many regions and how many countries per region we have ?

```{code-cell} ipython3
print(len(countries['name'].unique()))
len(countries['region'].unique())
```

### attach a region to each event

your mission is to now show the same graphs, but we want to reflect the relative usage of each region, so we want to [split each bar into several colors, one per region see expected result below](#label-events-output)

+++

most likely your first move is to tag all events with a `region` column

remember that you can `pandas.merge` two data frames

```{code-cell} ipython3
# on merge events_beg_sort et countries
df_final = pd.merge(events_beg_sort, countries, left_on='country', right_on='name')
df_final.head()
```

### visu by period by region

you can now produce [the target figures, again they look like this](#label-events-output)

remember that missing values can be filled with the `fillna` method and that timedelta can be computed with `pandas.Timedelta`

```{code-cell} ipython3
# on fait d'abord le code juste pour les semaines

df_week_region = df_final.groupby(['week', 'region'])['duration'].sum().map(convert_timedelta_to_hours) # on applique tout ce qu'on a fait précédemment
df_week_region.head()
```

```{code-cell} ipython3
# on cherche à avoir un df à 3 colonne : week, region, duration
df_week_region_flat = df_week_region.reset_index() # week redevient une colonne
df_week_region_flat.columns = ['week', 'region', 'duration']
df_week_region_flat.head()
```

```{code-cell} ipython3
# on crée table pivot (quand j'ai cherché comment afficher barchat complexe c'est ce qu'il était recommandé) 
df_week_region_pivot = df_week_region_flat.pivot(index='week', columns='region', values='duration').fillna(0) # les infos manquantes sont remplacées par un 0
```

```{code-cell} ipython3
# on fait de même pour les mois et années
df_month_region = df_final.groupby(['month', 'region'])['duration'].sum().map(convert_timedelta_to_hours)
df_month_region_flat = df_month_region.reset_index()
df_month_region_flat.columns = ['month', 'region', 'duration']
df_month_region_pivot = df_month_region_flat.pivot(index='month', columns='region', values='duration').fillna(0)

df_year_region = df_final.groupby(['year', 'region'])['duration'].sum().map(convert_timedelta_to_hours)
df_year_region_flat = df_year_region.reset_index() #
df_year_region_flat.columns = ['year', 'region', 'duration']
df_year_region_pivot = df_year_region_flat.pivot(index='year', columns='region', values='duration').fillna(0)
```

```{code-cell} ipython3
# on affiche tout

fig, axes = plt.subplots(1, 3)

df_week_region_pivot.plot.bar(ax=axes[0], stacked=True) # stacked = True pour que les bars s'empilent
axes[0].set_xticks(range(0, len(df_week_region_pivot), 12)) # on ne prend qu'une semaine sur 12

df_month_region_pivot.plot.bar(ax=axes[1], stacked=True)
axes[1].set_xticks(range(0, len(df_month_region_pivot), 3)) # idem mais tous les 3 mois

df_year_region_pivot.plot.bar(ax=axes[2], stacked=True)
plt.show()
```

***
