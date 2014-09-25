dc-budget-treemap
=================

Treemap for DC Budget (currently showing 2014 data) in D3; obviously inspired by Mike Bostock

### Inspiration

Saw this [data](https://docs.google.com/spreadsheet/ccc?key=0AvcRuGkEq26jdEk2cjgwQ3pkSThQTlRUUTJ5T3BURGc#gid=0) from [Justin Grimes](http://justgrimes.com/) in [this article](http://greatergreaterwashington.org/post/17987/visualize-the-dc-budget/).

Decided I wanted to recreate (and eventually add functionality to this) via D3.

Found a [great example](http://bost.ocks.org/mike/treemap/) from Mike Bostock.

### Data

All in [this folder](https://github.com/drewbo/dc-budget-treemap/tree/master/data)

2014 data available for download here: http://www.dcfpi.org/wp-content/uploads/2013/03/FY2014-General-Fund-Budget-spreadsheet-for-web-FINAL-v.-2.xls

Decided it would be easiest to get the data to match the format used in that example rather than recreating.

Downloaded the above data from the Google Spreadsheet and made three changes before saving as a CSV:

- Renamed `agency` to `name`
- Converted all uppercase strings to capitalize first letter of each word
- Renamed `fy2013` to `value`

Used [csvkit](https://csvkit.readthedocs.org/en/0.9.0/index.html) and [jq](http://stedolan.github.io/jq/) to get the data in the proper format with the following commands:

First convert the csv to json

    csvjson
    
Then convert the numbers which are stored as strings to numbers

    jq '[.[] | {category: .category, name: .name, change: .change | tonumber, value: .value | tonumber}]'
    
Then group our objects together by their `category` and mimic the tree structure from the Bostock example

    jq '{name: "DC Budget 2013", children: [group_by(.category) | .[] | {name: .[0].category, children: .}]}'
    
Run as one command with some piping and saving:

    cat data.csv | csvjson | jq '[.[] | {category: .category, name: .name, change: .change | tonumber, value: .value | tonumber}]' | jq '{name: "DC Budget 2013", children: [group_by(.category) | .[] | {name: .[0].category, children: .}]}' > data.json
    

### Display

Made some minor changes to the original CSS and JS for display reasons

Added two lines of javascript to color the boxes, representing each agency's budget, based on their change from the previous year:

    var color = d3.scale.linear().range(['red','white','green']).domain([-1,0,1])
    .style("fill", function(d) { return color(d.change / 100 || 0); })
    
Current state is hosted [here](http://drewbo.com/dc-budget-treemap/)

### Todos

- Color scale isn't great: improve
- Titles overrun their boxes: fix
- Probably could use hover functionality: make
- Maybe needs a shiny wrapper: create
- Switching between years: allow
