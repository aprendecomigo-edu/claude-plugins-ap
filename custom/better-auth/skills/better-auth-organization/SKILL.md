---
name: better-auth-organization
description: >
  Provides Better Auth organization plugin knowledge covering multi-tenant org creation, member management,
  invitation flows, team configuration, dynamic RBAC with custom roles, and lifecycle hooks.
  Triggers when working with organization-level features, role-based access control via Better Auth,
  team management, or member invitations. Note: this project currently uses custom multi-tenancy
  (school_roles, students, teachers, guardians tables) — consult this skill when considering
  migration to Better Auth's organization plugin.
---

# Organization Best Practices with Better Auth

This guide covers configuring multi-tenant organizations, managing members and teams, implementing role-based access control (RBAC), and setting up invitations using Better Auth's organization plugin.

## Quick Start

The setup requires three steps:

1. Add the `organization()` plugin to your server configuration
2. Include `organizationClient()` in your client setup
3. Run migrations to create necessary database tables

## Key Features

**Organization Creation**: Creators automatically receive the owner role. You can restrict creation to verified email users or implement tiered limits based on user plans.

**Member Management**: Add members server-side, assign multiple roles simultaneously, or use invitations for client-side additions. The system prevents removal of the last owner without first transferring ownership.

**Invitations**: Configure expiration periods (default 48 hours), set pending invitation limits per organization, and customize email templates. Shareable invitation URLs allow flexible distribution without automatic emails.

**Teams**: Enable team functionality with configurable limits for teams per organization and members per team. Users can set an active team for scoped operations.

**Dynamic Access Control**: Create custom roles beyond the default owner, admin, and member roles, with granular permission definitions for members, invitations, and other resources.

**Lifecycle Hooks**: Execute custom logic at various points — before/after organization creation, before deletion, and when members or invitations are created.

## Security Highlights

The last owner cannot be removed or leave an organization without transferring ownership first. Organization deletion removes all associated data. Consider disabling deletion or implementing soft delete via hooks. Invitations expire automatically and can only be accepted by the invited email address.
