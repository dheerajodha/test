<div align="center"><h1>Snapshot Controller</h1></div>

```mermaid
%%{init: {'theme':'forest',"themeVariables":{"max-width": "600px"}}}%%
flowchart TD
  %% Defining the styles
    classDef Red fill:#FF9999;
    classDef Amber fill:#FFDEAD;
    classDef Green fill:#BDFFA4;
  predicate((PREDICATE: <br>Snapshot got created OR <br> changed to Finished))
  %%%%%%%%%%%%%%%%%%%%%%% Drawing EnsureAllIntegrationTestPipelinesExist() function 
  %% Node definitions
  ensure1(Process further if: Snapshot testing <br>is not finished yet)
  areThereAnyITS{"Are there any <br>IntegrationTestScenario <br>present for the given <br>Application?"}
  doesITSHasEnvDefined{Does the <br>IntegrationTestScenario <br>has any environment <br>defined in it?}
  skipCreatingTestPLR(Skip creating Test PLR for this ITS,<br> as it will be created by binding controller)
  createNewTestPLR(<b>Create a new Test PipelineRun</b> for each <br>of the above ITS, if it doesn't exists already)
  markSnapshotInProgress(<b>Mark</b> Snapshot's Integration-testing <br>status as 'InProgress')
  fetchAllRequiredITS("Fetch all the required <br>(non-optional) IntegrationTestScenario <br>for the given Application")
  encounteredError1{Encountered error?}
  markSnapshotInvalid1(<b>Mark</b> the Snapshot as Invalid)
  IsAtleast1RequiredITS{Is there atleast <br>1 required ITS?}
  markSnapshotPassed(<b>Mark</b> the Snapshot as Passed)
  continueProcessing1(Controller continues processing...)
  %% Node connections
  predicate              ---->    |"EnsureAllIntegrationTestPipelinesExist()"|ensure1
  ensure1                -->      areThereAnyITS
  areThereAnyITS         --Yes--> doesITSHasEnvDefined
  areThereAnyITS         --No-->  fetchAllRequiredITS
  doesITSHasEnvDefined   --Yes--> skipCreatingTestPLR
  doesITSHasEnvDefined   --No-->  createNewTestPLR
  skipCreatingTestPLR    -->      fetchAllRequiredITS
  createNewTestPLR       -->      markSnapshotInProgress
  markSnapshotInProgress -->      fetchAllRequiredITS
  fetchAllRequiredITS    -->      encounteredError1
  encounteredError1      --No-->  IsAtleast1RequiredITS
  encounteredError1      --Yes--> markSnapshotInvalid1
  IsAtleast1RequiredITS  --Yes--> continueProcessing1
  IsAtleast1RequiredITS  --No-->  markSnapshotPassed
  markSnapshotPassed     -->      continueProcessing1
  %%%%%%%%%%%%%%%%%%%%%%% Drawing EnsureGlobalCandidateImageUpdated() function 
  %% Node definitions
  ensure2(Process further if: Component is not nil & <br>Snapshot testing succeeded & <br>Snapshot was not created by <br>PAC Pull Request Event)
  updateContainerImage("<b>Update</b> the '.spec.containerImage' field of the given <br>component with the latest value, taken from <br>given Snapshot's .spec.components[x].containerImage field")
  updateLastBuiltCommit("<b>Update</b> the '.status.lastBuiltCommit' field of the given <br>component with the latest value, taken from <br>given Snapshot's .spec.components[x].source.git.revision field")
  %% Node connections
  predicate             ----> |"EnsureGlobalCandidateImageUpdated()"|ensure2
  ensure2               -->    updateContainerImage
  updateContainerImage  -->    updateLastBuiltCommit
  %%%%%%%%%%%%%%%%%%%%%%% Drawing EnsureAllReleasesExists() function 
  %% Node definitions
  ensure3(Process further if: Snapshot is valid & <br>Snapshot testing succeeded & <br>Snapshot was not created by <br>PAC Pull Request Event)
  fetchAllReleasePlans(Fetch ALL the ReleasePlan CRs <br>for the given Application)
  encounteredError31{Encountered error?}
  createRelease(<b>Create a Release</b> for each of the above <br>ReleasePlan if it doesn't exists already)
  encounteredError32{Encountered error?}
  markSnapshotInvalid3(<b>Mark</b> the Snapshot as Invalid)
  continueProcessing3(Controller continues processing...)
  %% Node connections
  predicate            ---->    |"EnsureAllReleasesExists()"|ensure3
  ensure3              -->      fetchAllReleasePlans
  fetchAllReleasePlans -->      encounteredError31
  encounteredError31   --No-->  createRelease
  encounteredError31   --Yes--> markSnapshotInvalid3
  createRelease        -->      encounteredError32
  encounteredError32   --No-->  continueProcessing3
  encounteredError32   --Yes--> markSnapshotInvalid3
  %%%%%%%%%%%%%%%%%%%%%%% Drawing EnsureCreationOfEnvironment() function 
  %% Node definitions
  ensure4(Process further if: Snapshot testing <br>is not finished yet)
  step1FetchAllITS(Step 1: Fetch ALL the IntegrationTestScenario <br>for the given Application)
  step2FetchAllEnv(Step 2: Fetch ALL the Environments <br>present in the same namespace)
  selectITSWithEnvDefined(For each of the IntegrationTestScenario form Step 1, <br>select the ones that have .spec.environment field defined. <br>And process them in the next steps)
  doesEnvAlreadyExists{"Is there any <br>environment (from Step 2), <br>that contains labels with names <br>of current Snapshot and <br>IntegrationTestScenario?"}
  continueProcessing4(Controller continues processing...)
  copyAndCreateEphEnv(For each IntegrationTestScenario, <br> copy the existing env definition from <br>their spec.environment field and use it to <br><b>create a new ephemeral environment</b>)
  createSEBForEphEnv(<b>Create a SnapshotEnvironmentBinding</b> <br>for the given Snapshot and the <br>above ephemeral environment)
  %% Node connections
  predicate               ---->    |"EnsureCreationOfEnvironment()"|ensure4
  ensure4                 -->      step1FetchAllITS
  step1FetchAllITS        -->      step2FetchAllEnv
  step2FetchAllEnv        -->      selectITSWithEnvDefined
  selectITSWithEnvDefined -->      doesEnvAlreadyExists
  doesEnvAlreadyExists    --No-->  copyAndCreateEphEnv
  doesEnvAlreadyExists    --Yes--> continueProcessing4
  copyAndCreateEphEnv     -->      createSEBForEphEnv
  %%%%%%%%%%%%%%%%%%%%%%% Drawing EnsureSnapshotEnvironmentBindingExists() function 
  %% Node definitions
  ensure5(Process further if: Snapshot is valid & <br>Snapshot testing succeeded & <br>Snapshot was not created by <br>PAC Pull Request Event)
  AnyExistingNonEphEnv{Any existing root <br>and non-ephemeral <br>environment?}
  AnyExistingSEB{Any existing-SEB <br>containing the current <br>environment and <br>application?}
  UpdateExistingSEB(<b>Update</b> the existing-SEB <br>with the given Snapshot's name)
  createSEBForNonEphEnv("<b>Create a new <br>SnapshotEnvironmentBinding</b> (SEB) <br>with the current env and given Snapshot")
  encounteredError5{Encountered error?}
  markSnapshotInvalid5(<b>Mark</b> the Snapshot as Invalid)
  continueProcessing5(Controller continues processing...)
  %% Node connections
  predicate             ---->    |"EnsureSnapshotEnvironmentBindingExists()"|ensure5
  ensure5               -->      AnyExistingNonEphEnv 
  AnyExistingNonEphEnv  --Yes--> AnyExistingSEB
  AnyExistingNonEphEnv  --No-->  continueProcessing5
  AnyExistingSEB        --Yes--> UpdateExistingSEB
  AnyExistingSEB        --No-->  createSEBForNonEphEnv
  UpdateExistingSEB     -->      encounteredError5
  createSEBForNonEphEnv -->      encounteredError5
  encounteredError5     --Yes--> markSnapshotInvalid5
  encounteredError5     --No-->  continueProcessing5
  %% Assigning styles to nodes
  class predicate Amber;
  class encounteredError1,encounteredError31,encounteredError32,encounteredError5 Red;
```

