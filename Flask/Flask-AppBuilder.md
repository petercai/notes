# Views
Each view is a Flask **blueprint** that will be created for you automatically by the framework. You will map your methods to routing points, and each method will be registered as a possible security permission if you want. Its constructor will register your exposed urls on flask as a **Blueprint**, as well as all security permissions that need to be defined and protected.
