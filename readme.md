# Grafana reporter <img style="float: right;" src="https://travis-ci.org/IzakMarais/reporter.svg?branch=master">

A simple http service that generates \*.PDF reports from [Grafana](http://grafana.org/) dashboards.

![demo](demo/report_v5.gif)

## Requirements

Runtime requirements

- `pdflatex` installed and available in PATH.
- a running Grafana instance that it can connect to. If you are using an old Grafana (version < v5.0), see `Deprecated Endpoint` below.

Build requirements:

- [golang](https://golang.org/)

## Getting started

### Build and run

Get the source files and dependencies:

    go get github.com/IzakMarais/reporter/...

Build and install `grafana-reporter` binary to `$GOPATH/bin`:

    go install -v github.com/IzakMarais/reporter/cmd/grafana-reporter

Running without any flags assumes Grafana is reachable at `localhost:3000`:

    grafana-reporter

Query available flags. Likely the only one you need to set is `-ip`. 

    grafana-reporter --help
    -apiKey string
          Grafana api key. Required (and only used) in command line mode.
    -apiVersion string
          Api version: [v4, v5]. Required (and only used) in command line mode, example: -apiVersion v5. (default "v5")
    -dashboard string
          Dashboard identifier. Required (and only used) in command line mode.
    -o string
          Output file. Required (and only used) in command line mode. (default "out.pdf")
    -template string
          Specify a custom TeX template file. Only used in command line mode, but is optional even there.
    -ts string
          Time span. Required (and only used) in command line mode. (default "from=now-3h&to=now")
    -grid-layout
          Enable grid layout (-grid-layout=1). Panel width and height will be calculated based off Grafana gridPos width and height.
    -ip string
          Grafana IP and port. (default "localhost:3000")
    -proto string
          Grafana Protocol. Change to 'https://' if Grafana is using https. Reporter will still serve http. (default "https://")
    -ssl-check
          Check the SSL issuer and validity. Set this to false if your Grafana serves https using an unverified, self-signed certificate. (default true)
    -templates string
          Directory for custom TeX templates. (default "templates/")


### Generate a dashboard report

#### Query parameters

The endpoint supports the following optional query parameters. These can be combined using standard
URL query parameter syntax, eg:

    /api/v5/report/{dashboardUID}?apitoken=12345&var-host=devbox

**Time span**: The time span query parameter syntax is the same as used by Grafana.
When you create a link from Grafana, you can enable the _Time range_ forwarding check-box.
The link will render a dashboard with your current time range.  
By default, the time range will be included as the report sub-title. 
Times are displayed using the reporter's host server time zone. 


**variables**: The template variable query parameter syntax is the same as used by Grafana.
When you create a link from Grafana, you can enable the _Variable values_ forwarding check-box.
The link will render a dashboard with your current variable values.

**apitoken**: A Grafana authentication api token. Use this if you have auth enabled on Grafana. 
Syntax: `apitoken={your-tokenstring}`. If you are getting `Got Status 401 Unauthorized, message: {"message":"Unauthorized"}`
error messages, typically it is because you forgot to set this parameter. 

**template**: Optionally specify a custom TeX template file.
Syntax `template=templateName` implies the grafana-reporter should have access to a template file on the server at `templates/templateName.tex`.
The `templates` directory can be set with a command line parameter.
See the LaTeX code in `texTemplate.go` as an example of what variables are available and how to access them.
Also see [this issue](https://github.com/IzakMarais/reporter/issues/50) for an example. 


### Command line mode

If you prefer to generate a report directly from the command line without running a webserver,
command line mode enables this. All flags related to command line mode are
prefixed with `` to distinguish them from regular flags:

    grafana-reporter -apiKey [api-key] -ip localhost:3000 -dashboard ITeTdN2mk -ts from=now-1y -o out.pdf

## Development

### Test

The unit tests can be run using the go tool:

    go test -v github.com/IzakMarais/reporter/...

or, the [GoConvey](http://goconvey.co/) webGUI:

    ./bin/goconvey -workDir `pwd`/src/github.com/IzakMarais -excludedDirs `pwd`/src/github.com/IzakMarais/reporter/tmp/
