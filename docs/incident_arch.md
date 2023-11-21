```mermaid
flowchart TB

subgraph prisonStaff[Prison Staff]
    h1[-Person-]:::type
    d1[Prison Staff \n with NOMIS account]:::description
end
prisonStaff:::person

prisonStaff--Raises Incident -->uiApplication
prisonStaff--Raises Incident in NOMIS -->oracleForms

subgraph incidentReportingService[Incident Reporting Service]
    subgraph uiApplication[UI Application]
        direction LR
        h2[Container: Node / Typescript]:::type
        d2[Delivers the content for managing incident reports]:::description
    end
    uiApplication:::internalContainer

    subgraph incidentReportingApi[Incident Reporting API]
        direction LR
        h5[Container: Kotlin / Spring Boot]:::type
        d5[Provides incident report \n functionality via a JSON API]:::description
    end
    incidentReportingApi:::internalContainer

    subgraph database[Incident Reporting Database]
        direction LR
        h6[Container: Postgres Database Schema]:::type
        d6[Stores incident reports, \n historical reports and reference data]:::description
    end
    database:::internalContainer

    uiApplication--Makes API calls to -->incidentReportingApi
    incidentReportingApi--Reads from and \n writes to -->database
end
incidentReportingService:::internalSystem

incidentReportingApi--Publishes incident \n created/updated events -->domainEvents
incidentReportingApi--Audits changes -->auditService

domainEvents<--Listens to incident \n events from NOMIS-->incidentReportingApi
    
subgraph otherServices[Other HMPPS Services]
    subgraph prisonApi[Prison API]
        direction LR
        h31[Container: Java/Kotlin / Spring Boot]:::type
        d31[Exposes NOMIS data]:::description
    end
    prisonApi:::internalContainer

    subgraph prisonerSearchApi[Prisoner Search API]
        direction LR
        h32[Container: Kotlin / Spring Boot]:::type
        d32[Prisoner Data Cache]:::description
    end
    prisonerSearchApi:::internalContainer
    
    prisonerSearchApi--Retrieves Prisoner data from  -->prisonApi

end
otherServices:::internalSystem


domainEvents<--Listens to incident events from DPS -->nomisUpdateApi

subgraph eventsSystem[Event / Audit Services]
    subgraph domainEvents[Domain Events]
        direction LR
        h61[Container: SNS / SQS]:::type
        d61[Pub / Sub System]:::description
    end
    domainEvents:::internalContainer
end
eventsSystem:::internalSystem

subgraph auditSystem[Event / Audit Services]
    subgraph auditService[Audit Service]
        direction LR
        h62[Container: Kotlin / Spring Boot]:::type
        d62[Receives and records audit events]:::description
    end
    auditService:::internalContainer
end
auditSystem:::internalSystem

prisonApi--Reads from and \n writes to -->nomisDb
nomisPrisonerApi--read and update -->nomisDb
incidentReportingApi--looks up prisoner information -->prisonerSearchApi

subgraph NOMIS[NOMIS]
    subgraph oracleForms[NOMIS front end]
        direction LR
        h91[Container: Weblogic / Oracle Forms]:::type
        d91[Java applet screens surfacing NOMIS data]:::description
    end
    oracleForms:::legacyContainer
    
    subgraph nomisDb[NOMIS Database]
        direction LR
        h92[Container: Oracle 11g Database]:::type
        d92[Stores core \n information about prisoners, \n prisons, finance, etc]:::description
    end
    nomisDb:::legacyContainer

    oracleForms-- read/write data to -->nomisDb
end
NOMIS:::legacySystem


%% Element type definitions

classDef person fill:green, color:#fff
classDef internalContainer fill:#1168bd
classDef migrationContainer fill:wheat
classDef legacyContainer fill:purple
classDef internalSystem fill:lightblue
classDef legacySystem fill:lightblue 
classDef otherHmppsSystem fill: lightblue, color:#fff, stroke-width:0px  

classDef type stroke-width:0px, color:#fff, fill:transparent, font-size:12px
classDef description stroke-width:0px, color:#fff, fill:transparent, font-size:13px
```