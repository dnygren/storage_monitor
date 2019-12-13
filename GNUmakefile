################################################################################
# GNUmakefile : GNU makefile for storage_monitor
#
# Created by: Dan Nygren
# Email: dan.nygren@gmail.com
#
#   Makefile for installing and uninstalling storage_monitor.
#
# CALLING SEQUENCE      make [target]
#
# EXAMPLES
#                       make dist (Make a distribution file)
#                       make install (Copies storage_monitor to where it's used)
#                       make systemd (Enables and starts systemd service)
#                       make uninstall (stops & disables systemd & uninstalls)
#
# TARGET SYSTEM         Linux
#
# DEVELOPMENT SYSTEM    Linux, GNU make
#
# CALLS
#
# CALLED BY             N/A
#
# INPUTS
#
# OUTPUTS
#
# RETURNS               Not Applicable
#
# ERROR HANDLING        If there is an error in a rule, make gives up on the
#                       current rule.
#
# WARNINGS             (1. Describe anything a maintainer should be aware of)
#                      (2. Describe anything a maintainer should be aware of)
#                      (N. Describe anything a maintainer should be aware of)
#
################################################################################

# Enter the xz compressed tarball filename to be created by "make dist"
DIST_TXZ := storage_monitor_dist.tar.xz

# Installation directory
INSTDIR := /opt/storage_monitor

# Executable
EXE := storage_monitor

# Name of the login
LOGIN := pi

# Name of the systemd service unit configuration file
SERVICE := storage_monitor.service

# Name of the systemd timer unit configuration file
TIMER := storage_monitor.timer

# Name of any additional groups associated with the service.
# The flag is included to avoid an error if there are no additional groups.
# The uucp group is needed to access "/dev/ttyXXX" on the Zedboard and Rincon
# Astro SDR.
#GROUPS := --groups uucp
# The Raspberry Pi uses the dialout group to access "/dev/ttyXXX"
#GROUPS := --groups dialout
# You can separate multiple groups with a comma, but you will get an error if
# both groups do not exist. Usually the uucp OR dialout group exists, not both.
#GROUPS := --groups uucp,dialout

# If the STAGEDIR variable is undefined (the empty string) it has no effect.
# When it is defined, it causes installations to be staged to temporary
# directories for testing or before manually moving them to their final place.
# Staged installations do not enable or start a service.
#STAGEDIR := ./tmp_staged
STAGEDIR :=

# ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
# ^^^^^^^^^^ Place code that may need modification above this point. ^^^^^^^^^^
# ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

# It is a good practice to always explicitly state which shell make uses
SHELL := /bin/sh

# tar create options
TARCOPTIONS := --create
TARCOPTIONS += --auto-compress
TARCOPTIONS += --verbose
TARCOPTIONS += --preserve-permissions
TARCOPTIONS += --file

# tar extract options
TARXOPTIONS := --extract
TARXOPTIONS += --auto-compress
TARXOPTIONS += --verbose
TARXOPTIONS += --preserve-permissions
# TARXOPTIONS += --strip-components=1

# Mode that the installed directories can be set to.
# User read, write, and aXcess the directory
# Group read and aXcess the directory
# Others read the directory
DIRMODE := u=rwx,g=rx,o=r

# Mode that the installed files can be set to.
# User read and write the file
# Group read the file
# Others read the file
FILEMODE := u=rw,g=r,o=r

# Full path to executable
EXECUTABLE := $(INSTDIR)/$(EXE)

#
# Rules
#

#
# Other rules
#

.PHONY: dist
dist:
# Wrap the script, GNUmakefile, systemd files, and README into an xz compressed
# distribution tar file.
	/bin/tar $(TARCOPTIONS) $(DIST_TXZ) \
            GNUmakefile $(EXE) $(EXE).* README

# ### Rule to clean out executable, object, dependency and *.PRE files ###
# The "-" at the beginning of the command tells GNU make ignore errors like
# file not present etc. There is no need to delete parent directories that
# were created with "mkdir -p", since they could have existed anyway.
.PHONY: clean
clean:
	-/bin/rm -f $(DIST_TXZ)

.PHONY: uninstall
uninstall:
# Note that ifeq ($(STAGEDIR),) tests for an empty value of $(STAGEDIR) whereas
# ifdef($(STAGEDIR)) would just test if the $(STAGEDIR) was ever defined.
ifeq ($(STAGEDIR),)
# The dashes in front of the stop and disable allow the makefile to continue
# with the -f forced removes in case the service never started.
	-/bin/systemctl stop $(TIMER)
	-/bin/systemctl stop $(SERVICE)
	-/bin/systemctl disable $(TIMER)
	-/bin/systemctl disable $(SERVICE)
	/bin/rm -f /etc/systemd/system/$(TIMER)
	/bin/rm -f /etc/systemd/system/$(SERVICE)
# Remove all directories with installation directory prefix.
	/bin/rm -rf $(INSTDIR)
else
	/bin/rm -rf $(STAGEDIR)
endif

.PHONY: install
install:
# Set owner and group of directory to the login
	/usr/bin/install --mode $(DIRMODE) --owner=$(LOGIN) --group=$(LOGIN) \
--directory $(STAGEDIR)$(INSTDIR)
# Install files and set owner and group to the login
	/usr/bin/install --mode $(DIRMODE) --owner=$(LOGIN) --group=$(LOGIN) \
--target-directory $(STAGEDIR)$(INSTDIR) $(EXE)
#	/bin/chown --recursive $(LOGIN) $(STAGEDIR)$(INSTDIR)
#	/bin/chgrp --recursive $(LOGIN) $(STAGEDIR)$(INSTDIR)

.PHONY: systemd
ifeq ($(STAGEDIR),)
# If no staging directory defined, set the systemd to enable and start.
ENABLESERVICE := /bin/systemctl enable $(SERVICE)
STARTSERVICE := /bin/systemctl start $(SERVICE)
ENABLETIMER := /bin/systemctl enable $(TIMER)
STARTTIMER := /bin/systemctl start $(TIMER)
else
# Else just pretend to have enabled and started the systemd service.
ENABLESERVICE := true
STARTSERVICE := true
ENABLETIMER := true
STARTTIMER := true
endif

systemd:
# Test if systemd-run (used to create and start a service) exists and is
# non-zero in size. If so, install the service.
# Enable, but do not start service
# Enable and start timer
	/usr/bin/test -s /usr/bin/systemd-run && \
/usr/bin/install -m $(DIRMODE) -d $(STAGEDIR)/etc/systemd/system && \
/usr/bin/install -m $(FILEMODE) $(SERVICE) $(STAGEDIR)/etc/systemd/system && \
$(ENABLESERVICE) && \
/usr/bin/install -m $(FILEMODE) $(TIMER) $(STAGEDIR)/etc/systemd/system && \
$(ENABLETIMER) && $(STARTTIMER)
	@echo "*** The systemd $(SERVICE) is invoked by a systemd timer. ***"
	@echo "*** Therefore $(SERVICE) has no [Install] section. ***"
