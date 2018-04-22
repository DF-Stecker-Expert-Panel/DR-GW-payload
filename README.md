# DR-GW-payload

## Edit the draft rfc
For this purpose we use markdown syntax as it is described here:
https://github.com/miekg/mmark/wiki/Syntax

There is some xml structure required in addition - to be continued...

## Compile draft rfc
As it is not too easy to create the xml form required by IETF here is a brief description of this process.

On your Linux Host (with docker installed) run:

    mkdir -p $HOME/.cache/xml2rfc
    sudo docker run --rm --user=$(id -u):$(id -g) -v $(pwd):/rfc -v $HOME/.cache/xml2rfc:/var/cache/xml2rfc -w /rfc paulej/rfctools md2rfc draft-df-stecker-payload-tetra-00.md

which does all the magic

## Upload to IETF data tracker
