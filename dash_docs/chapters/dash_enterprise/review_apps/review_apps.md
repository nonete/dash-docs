# Review Apps

Review Apps are temporary "pre-release" versions of your Dash apps automatically 
created by your CI platform when making a pull request. These apps also inherit the 
configuration options of their production equivalent,  ensuring parity between 
environments for testing. Review Apps enable you to quickly deploy and share 
proposed changes with the rest of your team before merging them into production. 
Once those changes go live, the Review Apps are deleted. 

In this chapter will cover:

* Requirements
* Platform and service configuration
* Helper script
* CircleCI use case