# FrameUp Github Shared WorkFlows

This repository holds the code to FrameUp's standard CI. It's design so make as simpler as possible the github ci pipelines, adding dotnet Build, Test, SonarQube analysis, Docker Images Build and Publish with a few configuration code.

## What is its composition ?

### Build and Tests
The build process contains all the base steps to build and test a dotnet application. It will:

    - Setup .NET with a parametrized dotnet version;
    - Restore nuget dependencies;
    - Publishing the binaries by using dotnet publish command;
    - Store artifacts to be used in next steps, if necessary;
    - Execute Tests and execute SonarQube Quality gate;

You can check for more details at [this file](https://github.com/Team-One-Pos-Tech/FrameUp.SharedWorkFlows/blob/main/.github/workflows/build.yml)

#### How to use it ?
To enable CI at you repository by using the shared workflows, you only need to create a github workflow file, at ´.github/workflows/{filename}.yml (we recomend to use build.yml as file name).
With that file setup, you just need to "use" the shared workflow, and set some required variables, as you can see at the exeample below:

```yaml
on: [ push ]
jobs:
  build-and-test:
    uses: Team-One-Pos-Tech/FrameUp.SharedWorkFlows/.github/workflows/build.yml@main
    with:
      solution-name: "FrameUp.ProcessService.sln"
      api-project-name: "FrameUp.ProcessService.Api"
      sonar-project-key: "Team-One-Pos-Tech_FrameUp.ProcessService"
    secrets:
      sonar-token: ${{secrets.SONAR_TOKEN}}
```

Note: Sonar requires a token in order to integrate with, in order to get is, please, [follows Sonar Documentation](https://docs.sonarsource.com/sonarqube-server/latest/user-guide/managing-tokens/#generating-a-token)


### Docker and Containerization

In order to facilitate this process, this CI does some assumptions based on our team usage. So, to avoid having different docker files, a base onde is generated dureing the CI execution.
The base dockerfile will use the following template:
```sh
    echo '
    FROM ${{inputs.docker-base-image}} AS base
    WORKDIR /app
    EXPOSE 80
    EXPOSE 443

    FROM base AS final
    WORKDIR /app/${{inputs.image-name}}
    COPY /release/output /app/${{inputs.image-name}}/
    ENTRYPOINT ["dotnet", "'${{inputs.api-entrypoint-binary}}'"]
    ' > Dockerfile
```

with that, we have a standart docker container with small friction.
As you may notice, this template needs some details to be fullfilled, so, in order to use it, you just need to set it with you application details, like the example bellow:

```yaml
  docker-setup:
    needs: build-and-test
    uses: Team-One-Pos-Tech/FrameUp.SharedWorkFlows/.github/workflows/dockerize.yml@main
    with:
      image-name: "team-one-pos-tech/frameup-process-service"
      api-entrypoint-binary: "FrameUp.ProcessService.Api.dll"
```

You can check all variables and details at the [file](https://github.com/Team-One-Pos-Tech/FrameUp.SharedWorkFlows/blob/main/.github/workflows/dockerize.yml)