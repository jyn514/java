# Copyright (c) 2017 Joshua Nelson
# Licensed under the GNU General Public License v3.0

# In short, you may modify, distribute, or copy this program,
# but you may not change this license and you MUST make any
# modifications available AS SOURCE CODE to the users of this program.

# See https://www.gnu.org/licenses/gpl-3.0.en.html for details.


# ADD THIS FIRST
# program name here (requires proper capitalization)
PROGRAM = HelloWorld

#### Note before further modification: ####

# Many of these variables are used further on,
# especially in recipies for $(ARCHIVE) and $(TEST),
# as well as pattern-matching.

# If you simply uncomment existing code, you'll be fine.

# However, you may wish to glance over the rest of this file
# before making changes.

#### If working with C++ #################

#SOURCE = $(PROGRAM).cpp
#COMPILED = $(PROGRAM)

#$(COMPILED): $(SOURCE)
#	@:$(call fail_if_not_defined, SOURCE)
#	$(CC) -o $(COMPILED) $(SOURCE)

#$(PROGRAM).s: $(SOURCE)
#	@:$(call fail_if_not_defined, SOURCE)
#	$(CC) -S $(SOURCE)

#assembly: $(PROGRAM).s


#### If working with Java ################

SOURCE = $(PROGRAM).java
COMPILED = $(PROGRAM).class
$(COMPILED): $(SOURCE)
	@:$(call fail_if_not_defined,SOURCE)
	$(JAVAC) $(JAVACFLAGS) $(SOURCE)

# nothing to do, class files are (in essence) assembly
# more info here: https://en.wikipedia.org/wiki/Java_bytecode
assembly:

###########################################

# For your own sanity, I recommend you keep this updated
define NAMED_TASKS =
backup clean clobber over all run test assembly clean_run
endef

all: $(COMPILED)
	@:$(call fail_if_not_defined,COMPILED)

# Change this to a goal to have it run when make is called without arguments
# You will likely want it to be $(COMPILED) or all
.DEFAULT_GOAL := tasks

# If you add named goals, add them here to avoid naming conflicts
.PHONY: clean clobber backup tasks over all check test archive assembly run clean_run self_check \
	_self_check_start _assert_all_variables _emergency_backup _do_all_tasks

# Files that will be used later; feel free to change these
# Note, your test directory should NOT be the same as your current directory,
# as this will overwrite your existing files

ARCHIVE_DIR = archives
# := means evaluate immediately instead of lazily
ARCHIVE := $(ARCHIVE_DIR)/$(PROGRAM)-$(shell date -I).tar.gz

TEST_DIR = testing
TEST_ARCHIVE = $(TEST_DIR)/test.tar.gz
TEST_SCRIPT = test.sh

# I strongly encourage adding more tests
define my_tests =
	make $(COMPILED)
	make backup
	cat $(README)
	./$(COMPILED) | tee output1
	./$(COMPILED) wrong number of args | tee output1
	./$(COMPILED) right number | tee output1
	make clean
endef

clean:
	rm -rf $(TEST_DIR)
	rm -f $(COMPILED) *.out *.o *.s *.class $(TEST_SCRIPT)

###################### ARE YOU SURE YOU WANT TO EDIT ? ########################

# Good luck! For help, see
# https://www.gnu.org/software/make/manual/html_node/index.html#Top
# or email me at 'jyn514 AT gmail DOT com'

# Makefile specific
# https://www.gnu.org/software/make/manual/html_node/Makefile-Conventions.html
SHELL = sh
export MAKEFLAGS += --warn-undefined-variables -S
.SUFFIXES:
.SUFFIXES: .c .o .cpp .java .class
# get current makefile
# assumes current dir has no spaces in name (if false, rethink your choices)
MAKEFILE := $(lastword $(MAKEFILE_LIST))

# Compilers and flags
CC = g++ -g -Wall -Wpedantic -Wextra -O2
JAVAC != which javac
# note: if using the (deprecated) gcj, these flags are invalid.
JAVAC += -g -Xlint:all -source 1.8 -target 1.8

# ignore case, undefined if nothing exists
# https://unix.stackexchange.com/questions/198254
ifneq ($(shell find -maxdepth 1 -iname 'readme*' | wc -l), 0)
	README != find -maxdepth 1 -iname 'readme*'
endif

# add more files on next line for them to be added automatically to archive
# Note: this MUST come after ARCHIVE, MAKEFILE, and README are defined
# TODO: make this compatible with BSD make

$(ARCHIVE): $(ARCHIVE_DIR) $(MAKEFILE) $(README) $(SOURCE)
	@:$(call fail_if_not_defined,ARCHIVE MAKEFILE README SOURCE, \
		Files starting with README will be added automatically)
	tar -cvzf $(ARCHIVE) \
		$(filter-out $(ARCHIVE_DIR) ,$^)

# Takes 2 params: variable(s) which must exist, seperated by spaces (required); and description (optional)
# https://stackoverflow.com/questions/10858261
fail_if_not_defined = \
    $(strip $(foreach arg,$1, \
        $(call __check_single_var_defined,$(arg),$(strip $(value 2)))))

__check_single_var_defined = \
    $(if $(value $1),, \
      $(error $1 undefined$(if $2,. Comment: $2)))

# pattern matching: java
$(filter .class,$(COMPILED)): %.class: %.java
ifneq ($(strip $^),)
	$(JAVAC) $<
endif

# pattern matching: all .o files require corresponding .c or .cpp files
$(filter .o,$(COMPILED)): %.o: %.cpp
ifneq ($(strip $^),)
	$(CC) -c $^ -o $@
endif

# Directories
$(ARCHIVE_DIR):
	mkdir $(ARCHIVE_DIR)

$(TEST_DIR):
	mkdir $(TEST_DIR)

# overwrites existing test archive
$(TEST_ARCHIVE): $(ARCHIVE) $(TEST_DIR)
	ln -f $(ARCHIVE) $(TEST_ARCHIVE)

define newline =


endef

$(TEST_SCRIPT): $(TEST_DIR)
	echo '#!/bin/sh' > $(TEST_SCRIPT)
	echo 'set -ev' >> $(TEST_SCRIPT)
	echo 'tar -C $(TEST_DIR) -xvzf $(TEST_ARCHIVE)' >> $
	echo 'cd $(TEST_DIR)' >> $(TEST_SCRIPT)
	printf '$(subst $(newline),\n,$(my_tests))\n' >> $(TEST_SCRIPT)
	chmod u+x $(TEST_SCRIPT)


archive: $(ARCHIVE)

backup: $(ARCHIVE)

clobber: clean
	rm -rf $(ARCHIVE_DIR)

over: clean all

run: $(COMPILED)
	@:$(call fail_if_not_defined,COMPILED)
	./$(COMPILED)


test: clean $(TEST_ARCHIVE) $(TEST_SCRIPT)
	$(srcdir)/$(TEST_SCRIPT)
	@echo Success!

# self_check is named seperately so that _do_all_tasks doesn't start an infinite loop
tasks:
	@printf "tasks: self_check "
	@echo $(NAMED_TASKS)

check: test

clean_run: clean run

self_check: _do_all_tasks clean
	tar -xvzf emergency_backup.tar
	rm emergency_backup.tar
	echo All good!

_self_check_start:
	@printf "Running test of $(MAKEFILE)...\n\n"

_assert_all_variables: _self_check_start
	@printf "Asserting variables exist... "
	@:$(call fail_if_not_defined,ARCHIVE ARCHIVE_DIR CC COMPILED JAVAC \
	MAKEFILE MAKEFLAGS MAKEFILE_LIST PROGRAM SHELL SOURCE TEST_ARCHIVE TEST_DIR TEST_SCRIPT)
	@echo Succeeded.

_emergency_backup: _assert_all_variables
	@echo "Backing up existing files..."
	touch emergency_backup.tar
	tar -czvf emergency_backup.tar .
	@printf "Succeeded.\n\n"

_do_all_tasks: _emergency_backup
	for task in $(NAMED_TASKS); do \
		echo Executing task $$task ...; \
		$(MAKE) -f $(MAKEFILE) $$task; \
		printf "Succeeded.\n\n"; \
	done
