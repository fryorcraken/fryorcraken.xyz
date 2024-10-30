# fryorcraken.xxy

## Clone

Don't forget the submodules

```
git clone --recurse-submodules git@github.com:fryorcraken/fryorcraken.xyz.git
```

or

```
git clone git@github.com:fryorcraken/fryorcraken.xyz.git
git submodule update --init --recursive
```

## Install Hugo Extended

```
CGO_ENABLED=1 go install -tags extended github.com/gohugoio/hugo@latest
```

## Run Local Server

`-D`: include drafts

```
hugo server -D 
```