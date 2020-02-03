---
title: Release Notes
owner: Platform Engineering (KSM Team)
---

<strong><%= modified_date %></strong>

<%# The below partial is in https://github.com/pivotal-cf/docs-partials %>

<%= partial vars.path_to_partials + '/services/beta-notice',
:locals => {:product_full => vars.product_full, :platform_name => vars.platform_name} %>

These are release notes for <%= vars.product_full %>.

##<a id="0-X-XX"></a> v0.6.XX

**Release Date:** XXXX XX, XXXX

<p class="note breaking"><strong>Breaking Change:</strong> This release is incompatible with the v0.5
  beta release of <%= vars.product_short %>.
  Before you install <%= vars.product_short %> v0.6, you must uninstall
  <%= vars.product_short %> v0.5 from <%= vars.ops_manager_full %>. After you install <%= vars.product_short %> v0.6,
  you must re-add your service offerings.
  For more information, see <a href="./upgrading.html">Upgrading KSM</a>.
  </p>

### Features

New features and changes in this release:

* The <%= vars.product_short %> CLI now supports showing the version of the client and the server daemon with `ksm version`.
* `ksm offer save` had an --update-manifest flag. That has been changed to --update.
* <%= vars.product_short %> now supports the Helm values.yaml global.imageRegistry pattern.
* `ksm offer save` does not require the helm chart tgz to be in the command line if that version of the chart is already uploaded. 
* `ksm offer save` validates that the helm chart version specified in ksm.yaml matches the 
version in Chart.yaml on the command or already uploaded to chart museum. 


### Known Issue

This release has the following issue:

+ If your chart uses custom resources, <%= vars.product_short %> might incorrectly report that a service instance
was created successfully before the custom resources are created.
This is because Kubernetes does not have a generic method to detect resource readiness.
Kubernetes primitives have known patterns for detecting readiness, but custom resource readiness is defined by its author.

In the unlikely event that this happens, wait until the custom resources are created before binding
a service instances to an app.
This mitigates any issues with missing details in the bind response.