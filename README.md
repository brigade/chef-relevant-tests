Chef-Relevant-Tests
=========

> "Finally, a Chef gem not named after something food-related!" –Everyone

Have you ever wished that you could be *just a little* smarter about which
tests you run when changing your Chef cookbooks? This is the gem for you!

This is built to scratch an itch we have at Brigade: our single Chef repo has
(as of writing) 19 test-kitchen suites, but many of them are not relevant to
the average commit. We want to encourage the addition of more integration test
coverage by reducing the per-commit cost of adding a new suite, so, for every
commit we want to filter out the unaffected test suites as much as possible.

`gem install chef-relevant-tests`

Usage
--------
`chef-relevant-tests [old git ref] [expander]`

This command will examine all sources of difference between HEAD and your
previous ref (currently only the `Berksfile.lock`) and runs the updated
cookbook versions though a list of expanders which convert the the version
differences into the names of test suites.

Currently, the following expanders exist:

### test-kitchen
This gem will expand the run lists in your `.kitchen.yml` and only run suites dependent on changed cookbooks.

```bash
# if you have GNU xargs:
chef-relevant-tests [old git ref] test-kitchen | xargs --no-run-if-empty bundle exec kitchen test

# if you don't:
tests=$(chef-relevant-tests [old git ref] test-kitchen)
if [ -z "$tests" ]; then
  echo "No tests to run. Sweet!"
  exit 0
fi
kitchen test $tests
```

Example
----------
```bash
» git diff HEAD^ -- Berksfile.lock
   [...]
-  base_cookbook (1.4.18)
+  base_cookbook (1.4.19)
   [...]

» cat .kitchen.yml
[...]
suites:
  - name: app-server-suite
    run_list: ["app_server::default"]    # app_server cookbook depends upon "base_cookbook"
  - name: mysql-suite
    run_list: ["mysql::install"]         # mysql cookbook also depends upon "base_cookbook"
  - name: base-cookbook-suite
    run_list: ["base_cookbook"]
  - name: other-suite
    run_list: ["other_cookbook"]         # does not depend upon base_cookbook

» chef-relevant-tests HEAD^ test-kitchen
app-server-suite mysql-suite base-cookbook-suite
```
Architecture
----------
This gem is meant to be extended by adding new `ChangeDetectors` (which convert
the diff between two git refs into cookbook names) or `Expanders` (which take
cookbook names and convert it into test suite names).

How does this compare to [Strainer][1]?
---------
Whereas Strainer meant to isolate individual test runs, this gem is meant to
ease the pain of a large Chef repository by culling the test suite down to just
the tests likely to have been affected by your change.

It would be possible to make a `Expander` which outputs cookbooks in the format
for Strainer, but this is not implemented yet. Feel free to create an issue for
this if you would like it!

[1]: https://github.com/customink/strainer
