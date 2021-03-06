# debugging: MOCHA='mocha --debug-brk'

JISON?=../node_modules/.bin/jison
MOCHA?=../node_modules/.bin/mocha
TESTDIR?=../../shexTest/ # needs trailing '/'
GENJSON?=$(join $(TESTDIR), bin/genJSON.js)
GENCorpus?=$(join $(TESTDIR), bin/genCorpus.js)
REPORTER?=dot # override with e.g. REPORTER=mocha-earl-reporter
ASSERTER?=http://www.w3.org/People/Eric/ericp-foaf#me
SUBJECT?=https://www.npmjs.com/package/shex
TEST_PREFIX?=https://raw.githubusercontent.com/shexSpec/shexTest/master/
RPRT_OPTS?=--reporter-options asserter=$(ASSERTER),subject=$(SUBJECT),testPrefix=$(TEST_PREFIX)
REPORTER1=mocha-earl-reporter $(RPRT_OPTS)

BRANCH?=$(shell git rev-parse --abbrev-ref HEAD) # e.g. master or ShEx1
HTTPTEST?="http://raw.githubusercontent.com/shexSpec/shex.js/$(BRANCH)/test/"

ValidationManifest=$(join $(TESTDIR), validation/manifest.jsonld) # isn't there an easier way to construct filenames?
ValidationCorpus=$(join $(TESTDIR), schemas/corpus.json) # isn't there an easier way to construct filenames?
SHEXSCHEMAS = $(wildcard $(join $(TESTDIR), schemas/*.shex))
JSONSCHEMAS = $(SHEXSCHEMAS:shex=json)

../packages/shex-parser/lib/ShExJison.js: ../packages/shex-parser/lib/ShExJison.jison
	$(JISON) $^ -o $@ -p lr
	perl -pi -e 's/loc: yyloc,$$/loc: lexer.yylloc,/' $@ # yyloc is reset by lex()->next()->test_match() [[ this.yylloc = {...} ]]

../packages/shape-map/lib/ShapeMapJison.js: ../packages/shape-map/lib/ShapeMapJison.jison
	$(JISON) $^ -o $@ -p lalr
	perl -pi -e 's/loc: yyloc,$$/loc: lexer.yylloc,/' $@

Parser-Writer-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js Parser-Writer-test.js
	$(MOCHA) -C -R $(REPORTER) Parser-Writer-test.js

Partition-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js Partition-test.js
	$(MOCHA) -C -R $(REPORTER) Partition-test.js

ShExPath-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js ShExPath-test.js
	$(MOCHA) -C -R $(REPORTER) ShExPath-test.js

Compilation-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js ../packages/shex-core/lib/ShExUtil.js Compilation-test.js
	$(MOCHA) -C -R $(REPORTER) Compilation-test.js

$(ValidationManifest): $(join $(TESTDIR), validation/manifest.ttl) $(GENJSON)
	$(GENJSON) $< -o $@ -e # suppress output if there are errors

$(ValidationCorpus): $(GENCorpus) $(JSONSCHEMAS)
	$(GENCorpus) $< -d $(join $(TESTDIR), schemas/) -o $@ -e # suppress output if there are errors

Validation-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js ../node_modules/@shexjs/core/lib/ShExValidator.js Validation-test.js $(ValidationManifest)
	$(MOCHA) -C -R $(REPORTER) Validation-test.js

Map-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js Map-test.js
	$(MOCHA) -C -R $(REPORTER) Map-test.js

cli-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js cli-test.js
	$(MOCHA) -C -R $(REPORTER) cli-test.js

browser-test: ../packages/shex-parser/lib/ShExJison.js ../packages/shape-map/lib/ShapeMapJison.js browser-test.js
	$(MOCHA) -C -R $(REPORTER) browser-test.js

map-extensions-test: ../packages/shex-parser/lib/ShExJison.js map-extensions-test.js
	$(MOCHA) -C -R $(REPORTER) map-extensions-test.js

hashmap-extension-test: ../packages/shex-parser/lib/ShExJison.js hashmap-extension-test.js
	$(MOCHA) -C -R $(REPORTER) hashmap-extension-test.js

regex-extension-test: ../packages/shex-parser/lib/ShExJison.js regex-extension-test.js
	$(MOCHA) -C -R $(REPORTER) regex-extension-test.js

test: Parser-Writer-test Partition-test ShExPath-test Compilation-test Validation-test Map-test cli-test browser-test map-extensions-test hashmap-extension-test regex-extension-test.js

