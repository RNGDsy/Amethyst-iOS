SOURCEDIR := src
OUTPUTDIR := build
CLASSDIR := $(OUTPUTDIR)/classes
STAGEDIR := $(OUTPUTDIR)/stage

SOURCES := $(shell find $(SOURCEDIR) -type f -name '*.java')
CLASSES := $(SOURCES:$(SOURCEDIR)/%.java=$(CLASSDIR)/%.class)

TARGETS := $(shell find $(SOURCEDIR) -mindepth 1 -maxdepth 1 -type d)
TARGETS := $(TARGETS:$(SOURCEDIR)/%=$(OUTPUTDIR)/%.jar)

LWJGL_DIRS := libs/lwjgl-3.3.3 libs/lwjgl-3.4.1
LWJGL_JARS := $(shell find $(LWJGL_DIRS) -type f -name 'lwjgl*.jar' 2>/dev/null)
GSON_JARS := $(shell find libs -type f -name 'gson*.jar' 2>/dev/null)

JCFLAGS := -cp "$(subst $() $(),:,$(wildcard $(SOURCEDIR)/*/):$(wildcard libs/*/*.jar):$(LWJGL_JARS):$(GSON_JARS))" -XDignore.symbol.file

all: $(TARGETS)

check_empty_classes:
	@set -e
	@echo "Checking for empty class files"
	@EMPTY_CLASSES="$$(find $(CLASSDIR) -name '*.class' -size 0)"; \
	if [ -n "$$EMPTY_CLASSES" ]; then \
		echo "Error: found empty class files: $$EMPTY_CLASSES"; \
		exit 1; \
	fi

$(CLASSDIR)/%.class: $(SOURCEDIR)/%.java
	@set -e
	@mkdir -p $(dir $@)
	@echo "Compiling $<"
	@$(BOOTJDK)/javac $(JCFLAGS) -d $(CLASSDIR) $<

$(OUTPUTDIR)/launcher.jar: $(CLASSES) check_empty_classes
	@set -e
	@echo "Creating $@"
	@rm -rf $(STAGEDIR)
	@mkdir -p $(STAGEDIR)
	@cp -R $(CLASSDIR)/* $(STAGEDIR)/ 2>/dev/null || true
	@$(BOOTJDK)/jar -cf $@ -C $(STAGEDIR) .

$(OUTPUTDIR)/lwjgl.jar: $(CLASSES) check_empty_classes
	@set -e
	@echo "Creating $@"
	@if [ -z "$(strip $(LWJGL_JARS))" ]; then \
		echo "Error: no LWJGL jars found in $(LWJGL_DIRS)"; \
		exit 1; \
	fi
	@rm -rf $(STAGEDIR)
	@mkdir -p $(STAGEDIR)
	@for j in $(LWJGL_JARS); do \
		tmpdir="$$(mktemp -d)"; \
		$(BOOTJDK)/jar -xf "$$j" -C "$$tmpdir" 2>/dev/null || (cd "$$tmpdir" && $(BOOTJDK)/jar -xf "$$j"); \
		cp -R "$$tmpdir"/* $(STAGEDIR)/ 2>/dev/null || true; \
		rm -rf "$$tmpdir"; \
	done
	@rm -rf $(STAGEDIR)/META-INF
	@$(BOOTJDK)/jar -cf $@ -C $(STAGEDIR) .

$(OUTPUTDIR)/patchjna_agent.jar: $(CLASSES)
	@set -e
	@rm -rf $(STAGEDIR)
	@mkdir -p $(STAGEDIR)
	@cp -R $(CLASSDIR)/* $(STAGEDIR)/ 2>/dev/null || true
	@mv $(STAGEDIR)/com/sun/jna/Platform.class $(STAGEDIR)/com/sun/jna/Platform.class.patch
	@echo "Creating $@"
	@$(BOOTJDK)/jar -cfm $@ $(SOURCEDIR)/patchjna_agent.txt -C $(STAGEDIR) .

clean:
	rm -rf $(OUTPUTDIR)

.SUFFIXES: .java
.PHONY: all clean check_empty_classes
