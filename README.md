Howto test with catalog_preview  without a real master
======================================================

We have one environment per branch. We would like to test production against
a new feature environment.

Git branch master = production environment.

We would like to see what this means to our systems without involving
the real master.

To compile a catalog we require the ENC data and the facts. Both of them
are saved in the yamldir on the master.

It is assumend you have your hieradata also in your environment.

so you can ...

* copy the masters yamldir (`puppet master --configprint yamldir`) to 
  `mock_yaml` 
* adjust the files to your liking , maybe remove
  * ... sensitive data ,
  * ... redundant hosts
* install the `catalog_preview` module: `puppet module install puppetlabs-catalog_preview`
  * it's not required to do it as root!
  * it will save it somehwere in your homedir (`puppet master --configprint basemodulepath`)
* checkout baseline (master branch) and preview (feature branch) to a environments folder, so you get:
  * `environments/production/...` and 
  * `environments/feature/...` 
* run `puppet preview` with the following additional parameters:
  * `--environmentpath $(pwd)/environments`
  * `--yamldir $(pwd)/mock_yaml`
  * `--baseline_environment=production`
  * `--preview_environment=feature`
  * ... additional params required if you have hieradata (`hiera_config` 
    param and a corresponding `hiera.yaml`) !

Example
-------

File/directory structure for this example:

```
./environments/feature/manifests/site.pp
./environments/production/manifests/site.pp
./mock_yaml
./mock_yaml/node/mock.node.yaml
./mock_yaml/node/mock1.node.yaml
./mock_yaml/facts/mock.node.yaml
./mock_yaml/facts/mock1.node.yaml
```


Preview:

```

#> /opt/puppetlabs/bin/puppet preview \
  --environmentpath $(pwd)/environments \
  --yamldir $(pwd)/mock_yaml \
  --baseline_environment=production \
  --preview_environment=feature \
  --view overview \
  mock.node mock1.node

Stats
  Total number of nodes: 2, 100.0%
  Conflicting..........: 2, 100.0%
  Compliant............: 0,   0.0%
  Equal................: 0,   0.0%

Changes per Resource Type
  Notify
    title: 'määääääh' (missing, conflicting) at: /home/mock/tmp/puppettest/environments/production/manifests/site.pp:1 on mock.node, mock1.node
    title: 'muhhh' (added, compliant) at: /home/mock/tmp/puppettest/environments/feature/manifests/site.pp:1 on mock.node, mock1.node

Changes of Edges
  added_edges
    Class[main] => Notify[muhhh] on nodes mock.node, mock1.node
  missing_edges
    Class[main] => Notify[määääääh] on nodes mock.node, mock1.node

Top ten nodes with most issues
  node name   errors  warnings   diffs
  ---------- -------- -------- --------
  mock.node         0        0        2
  mock1.node        0        0        2

```

