# Ghidra

## Installation (Linux & OSX)

## Package Installer:

`brew install ghidra`

## Manual Installation

{% embed url="https://htmlpreview.github.io/?https://github.com/NationalSecurityAgency/ghidra/blob/Ghidra_10.2.2_build/GhidraDocs/InstallationGuide.html#Platforms" %}
Reference
{% endembed %}

### Instructions:

* Linux and macOS (OS X): Extract the JDK distribution (.tar.gz file) to your desired location, and add the JDK's bin directory to your PATH:
  1.  Extract the JDK:

      > _tar xvf \<JDK distribution .tar.gz>_
  2.  Open \~/.bashrc with an editor of your choice. For example:

      > _vi \~/.bashrc_
  3.  At the very end of the file, add the JDK bin directory to the PATH variable:

      > _export PATH=\<path of extracted JDK dir>/bin:$PATH_
  4. Save file
  5. Restart any open terminal windows for changes to take effect

#### Grab the JDK here:

{% embed url="https://github.com/NationalSecurityAgency/ghidra/releases" %}
GitHub Repository
{% endembed %}

## Dark Mode

{% embed url="https://github.com/zackelia/ghidra-dark" %}
