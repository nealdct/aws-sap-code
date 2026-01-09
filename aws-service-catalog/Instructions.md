Original AWS tutorial: https://docs.aws.amazon.com/servicecatalog/latest/adminguide/getstarted.html

## Create a Portfolio

Name: Engineering Tools
Description:  Sample portfolio that contains a single product.
Owner: IT (it@example.com)

## Create a Product

Name: Linux Desktop
Description: Cloud development environment configured for engineering staff. Runs AWS Linux.
Owner: IT
Upload template: cfn-template.json
Version title: v2.0
Description: Base Version
Support contact: ITSupport@example.com
Support description: Contact the IT department for issues deploying or connecting to this product.

Add product to portfolio

## Add a template constraint

Product: Linux Desktop
Contraint type: Template
Description: Small instance sizes
Add JSON from: template-constraint.json

## Add a Launch Constraint

Create IAM policy:
    JSON: Add code from launch-constraint.json
    Name: linuxDesktopPolicy

Create IAM role:
    Trusted entity: AWS Service > Service Catalog
    Policy: linuxDesktopPolicy
    Name: linuxDesktopLaunchRole

Add constraint in Service Catalog:
    Constraint type: Launch
    IAM role: linuxDesktopLaunchRole

## Grant end users access

Create IAM group:
    Name: Endusers
    Policy: AWSServiceCatalogEndUserFullAccess

Then add relevant users to the group

Provide access to portfolio by adding Endusers group to portfolio in the "Groups, roles, and users" tab


## Test access

Sign in as user with end user access and attempt to launch product

