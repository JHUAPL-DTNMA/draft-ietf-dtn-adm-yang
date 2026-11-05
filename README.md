Prerequisites to building can be installed on Ubuntu with:
```
sudo apt-get install -y cmake python3-pip python3-wheel ruby xmlstarlet aspell
sudo pip3 install abnf xml2rfc pyang mako
sudo gem install cddl
```
and then the document can be built with
```
cmake -S . -B build/default
cmake --build build/default
```
finally opened with
```
xdg-open build/default/draft-*.html
```

The YANG plugin and module can be used similar to:
```
PYANG_PLUGINPATH=pyang YANG_MODPATH=modules pyang modules/amm.yang --lint
```
and
```
PYANG_PLUGINPATH=pyang YANG_MODPATH=modules pyang examples/example-adm.yang --lint
```

To check and reformat one of the ADMs for this document, clone the ACE repo locally and run a merging tool similar to:
```
meld modules/ietf-dtnma-agent.yang <(PATH=~/pyang/bin:$PATH PYTHONPATH=~/anms-ace/src:~/pyang YANG_MODPATH=modules pyang --strict --ietf -t adm-add-enum -f yang --canonical modules/ietf-dtnma-agent.yang)
```
