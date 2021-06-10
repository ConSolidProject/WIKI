# WIKI
This repository contains general information about the ConSolid Project, its goals and current status. 

## Design Patterns
The ConSolid project is a research project for "federated Common Data Environments" (CDEs). The project architecture is governed by a limited set of parameters, intended to maximise flexibility while nevertheless providing a practical framework for handling federated building data. These parameters share common patterns, but can be considered independently. Hence, should the need rise to change one of the parameters, or should other projects consider to adopt certain patterns but leave others out, this is perfectly possible. Therefore, we describe below the core ideas behind a parameter, and some possible adaptations.

* The framework supports heterogeneous datasets
CORE: A federated project may consist of multiple heterogeneous datasets (3D geometry, imagery, point clouds, semantics), but the 'semantic glue' between those datasets will be based on Semantic Web technologies (RDF). Ideally, semantics are expressed in RDF as well, making the project entirely "data-based" (i.e. not document-based/siloed). 
FLEXIBILITY: In theory, however, the concept of a federated CDE is not dependent on the nature of the semantics. Hence, (document-based) data formats such as IFC SPFF can still be used in a federated environment, although other Semantic Web benefits (e.g. federated querying and rule checking in a domain-agnostic fashion) will not be possible or at least be more difficult to achieve. As in future phases of the project, use cases will be developed that rely upon external APIs (which most likely will not be RDF-based), the ability to support both data- and file-based solutions was taken into account from the beginning.

* The framework is modular both regarding data models and tool stack
CORE: As it should be possible for each project to determine which data models are needed, the framework cannot impose the use of specific data models. It can, however, advise certain patterns data models should comply to, as to make them optimally function in the framework. In this case, the modular LBD ontologies from the W3C LBDCG are advised to be used, as they are small in size and are designed to be combined with other ontologies.
FLEXIBILITY: Although we consider the LBD ontologies to be more flexible and user-friendly, many of them are derived from corresponding IFC concepts. An ecosystem primarily based on IfcOWL would therefore function as well - albeit more difficult to built services upon due to its size and limited modularity.

* Project data can be federated
CORE: The use of data federation technologies such as the Solid specifications allows to decouple data and applications. This essentially means that applications do not keep a database themselves, but all information they use is gathered via API requests, either to access-restricted data vaults (Solid), open data APIs or other services. Firstly, this allows for a uniform way of data handling. Secondly, by considering an AEC project as a set of APIs, a project becomes inherently federated. When decentralised authentication is used, such APIs can be access-restricted. The project envisages a software deliverable that allows individual stakeholders in a project to manage their data themselves instead of uploading to a central server and control all their projects in a self-hosted data environment.
FLEXIBILITY: The core idea behind data decentralisation and federation is that it does not really matter where data is stored. We imagine self-hosting would be the most interesting, but in exactly the same way users can opt to outsource the hosting process to specialised firms. Moreover, a project consortium can still decide to centralise all information on a remote server, and still use all the other features of the ecosystem.

* Services are federated, yet can be organised in one GUI and exchange data
CORE: Web services are federated, following the widely implemented idea of Web APIs. This project will make use of these "headless" web services, but will also provide a framework for data manipulation that require an interaction between a human actor and multiple microservices that need to exchange data with one another. Therefore, we base upon the module federation technologies provided in the recent Webpack 5. This allows a loose-coupling between separate GUI applications within a single browser window. Using configuration files targeted at a specific use case, it becomes possible to combine different modules and generate a dedicated GUI that relies on multiple remote modules and headless APIs. Storing such configurations in federated project vaults (Solid PODs) will allow to relate specific configurations to specific project tasks. 
FLEXIBILITY: The core idea behind this is maximal flexibility for GUI generation and improved development speed. The design patterns to embed a federated module into the framework will be thoroughly documented. GUIs can be developed in any major front-end development frameworks or libraries (Angular, React, Vue ...), as the bundling of modules is an independent step (JS/HTML/CSS).

A visual overview of these patterns can be seen in the following image: 

![consolid_architecture](https://user-images.githubusercontent.com/33063548/121509695-35130780-c9e7-11eb-8e97-5df464ceb2f2.png)

## Current implementations
### Data exchange between modules
The prototypical code base is stored at https://github.com/ConSolidProject. It contains the code for the front-end building "container", and some plugin applications for testing purposes. By creating such plugin applications, we expect the minimal requirements for data exchange between microfrontends will become clear. At this point, the following data (get, set) is made available via the container module: 

* "artefacts of interest" (e.g. selection): By exchanging artefacts of interest, a viewer can visualise the objects that are the result of a query or SHACL validation. These artefacts are stored in a list, as separate objects. This allows to store core data about the artefact (e.g. the "root object") but also specify other data such as geometric GUIDs related to that object. It is to be discussed whether such additional data is necessary or whether an application should fetch/reconstruct this data itself. In the latter case, only a URI of the "root" that represents an real-world object will be exchanged.
* current project: a plugin must be able to know a project in order to propagate through it and find relevant resources. The current structure of a project is described at https://github.com/ConSolidProject/sample_pod_data. The structure of the exchanged "currentproject" object is still preliminary at this point. Essentially, it could consist of only the project POD URL, from where an application can fetch the necessary data. But it could also be a metadata graph (DCAT) that has already pointers to project resources, so they need not be fetched every time a separate application needs it (to avoid unnecessary fetching). A third option is to store certain data in the browser session, so plugins can use them when necessary. To be further explored.
* Active graphs / active documents. In the current infrastructure, "active" resources are exchanged between modules. This makes it possible to "activate" resources in a project tree, so e.g. a viewer can be instantiated to show any 3D geometry or point cloud of an active document, or a federated query engine knows what RDF graphs it should include in its queries.
* session object: currently, a Solid session object is exchanged between modules. This session may also be retrieved directly from the browser. However, it is expected that this will be obsolete in a Solid-based configuration, as the session is already stored in the browser as a browser session.

### Front-end bundling
The infrastructure makes use of the novel "module federation" functionality provided in Webpack 5. This allows to bundle independent modules, which may be standalone applications on another server, into a single environment. As described above, this is possible independently from the framework this front-end is programmed in. We think this offers opportunities for an open network of Linked Building Data applications that may interact with one another. We are developing a low-threshold plugin system that allows to bundle specific interfaces on runtime, based on a JSON-based configuration. 

Think of it like a shopping list for modules: for example, a developer wants to make a semantic enrichment module that allows, in a user-friendly way, to enrich the model with damage data. In addition to her own core-module, she figures an end user will probably need a geometric viewer module, a project tree module and a query module. Using a (to be refined) configuration file, those elements can be fetched from existing applications. When these applications follow the same design patterns, a module can be easily replaced with another one that seems better fit for the purpose. 

The current config file structure (see below for an example) is rather a proof-of-concept. Goals are to be able to generalise this and find better semantic structures to represent the modules (JSON-LD). 

```
{
  "viewer1": {
    "component": "RemoteComponent",
    "x": 0,
    "y": 0,
    "w": "100%",
    "h": "92%",
    "fixed": true,
    "props": {
      "system": {
        "url": "http://localhost:4002/remoteEntry.js",
        "scope": "viewer",
        "module": "./index"
      }
    }
  },
  "manager": {
    "component": "RemoteComponent",
    "x": 900,
    "y": 100,
    "w": 500,
    "h": 600,
    "props": {
      "system": {
        "url": "http://localhost:4003/remoteEntry.js",
        "scope": "querymodule",
        "module": "./index"
      }
    }
  },
  "plugins": {
    "component": "SidePlugins",
    "props": {
      "moduleId": "imgviewer1"
    },
    "x": 0,
    "y": 0,
    "w": 500,
    "h": "92%",
    "children": {
      "template": { "component": "TemplatePlugin" },
      "remote1": {
        "component": "RemoteComponent",
        "props": {
          "system": {
            "url": "http://localhost:4001/remoteEntry.js",
            "scope": "projectmanager",
            "module": "./index"
          }
        }
      }
    }
  }
}

```

We imagine a graphical infrastructure to be possible where people can create dedicated configurations for dedicated purposes. Some kind of LBD app store, which can be used by both developers and end-users. In the case of end users, specific tasks in the (federated?) project can be linked to a particular config file (e.g. for model checking). This "app store" is, however, a long-term goal.

NOTE: at this point, a config constitutes of a single page render for interaction with a federated model, which is probably enough for this purpose. This concept of configs can be abstracted even further, however, to allow the entire front-end to be configured this way. 



