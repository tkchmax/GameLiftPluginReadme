# Amazon GameLift Plugin for Unreal Engine

## Overview

Amazon GameLift provides tools for preparing your multiplayer games and custom game servers to run on the GameLift
service. The GameLift SDKs contain libraries needed to enable game clients and servers to communicate with the GameLift
service. The Amazon GameLift Plugin for Unreal Engine makes it easier to access GameLift resources and integrate GameLift into
your Unreal Engine game. You can use the Plugin for Unreal Engine to access GameLift APIs and deploy AWS CloudFormation templates for
common gaming scenarios. Pre-built sample scenarios include:

* Auth Only — This scenario creates a game backend service that performs only player authentication and no game server
  capability. It creates a Cognito user pool to store player authentication information, as well as an API gateway REST
  endpoint backed up AWS Lambda handlers to start a game and view game connection information. The Lambda handler always
  returns a 501 Error (Unimplemented).

* Single-Region Fleet — This scenario creates a game backend service with a single GameLift fleet. After player
  authenticates and starts a game (with a `POST` request to `/start_game`), a AWS Lambda handler searches for an
  existing viable game session with an open player slot on the fleet via `gamelift::SearchGameSession`. If an open slot
  is not found, the Lambda creates a new game session via `gamelift::CreateGameSession`. Once game start is request, the
  game client should poll the back end (with `POST` requests to `/get_game_connection`) to receive a viable game
  session.

* Multi-Region Fleets with Queue and Custom Matchmaker — In this scenario, Amazon GameLift queues are used in
  conjunction with a custom matchmaker. The custom matchmaker forms matches by grouping up the oldest players in the
  waiting pool. The customer matchmaker does not consider other factors like skills or latency. When there are enough
  players for a match, the Lambda calls `GameLift:StartGameSessionPlacement` to start a queue placement. Once the
  placement is done, GameLift publishes a message to the SNS topic in the backend service, which triggers a Lambda
  function to store the placement details along with the game conection details to a DynamoDB table. Subquent
  GetGameConnection calls would read from this table and return the connection information to the game client.

* SPOT Fleets with Queue and Custom Matchmaker — This This scenario is the same as Multi-Region Fleets with Queue and
  Custom Matchmaker except it configures three fleets. Two of the fleets are SPOT fleets containing nuanced instance
  types to provide durability for SPOT unavilabilities; the third fleet is an ON_DEMAND fleet to serve as a backup in
  case the other SPOT fleets go unviable. Using a GameLift queue can keep availability high and cost low. For more
  information and best practices about queues,
  see [Setting up GameLift queues for game session placement](https://docs.aws.amazon.com/gamelift/latest/developerguide/queues-intro.html)
  .

* FlexMatch — This scenario uses GameLift FlexMatch, customizable matchmaking service for multiplayer games. On
  StartGame requests, a Lambda creates matchmaking ticket via `gamelift:StartMatchmaking`, and a separate lambda listen
  to FlexMatch events similar to the queue example above. This deployment scenario also uses a low frequency poller to
  describe incomplete tickets via `gamelift::DescribeMatchmaking`. The incomplete tickets are periodically described so
  they are not discarded by GameLift. This is a best practice recommended
  by [Track Matchmaking Events](https://docs.aws.amazon.com/gamelift/latest/flexmatchguide/match-client.html#match-client-track)
  . For more information on FlexMatch,
  see [What is GameLift FlexMatch?](https://docs.aws.amazon.com/gamelift/latest/flexmatchguide/match-intro.html)

Each sample scenario uses an AWS CloudFormation template to create a stack with all of the resources needed for the
sample game. You can remove the resources by deleting the corresponding AWS CloudFormation stack.

The Amazon GameLift Plugin for Unreal Engine also includes a sample game you can use to explore the basics of integrating your
game with Amazon GameLift.

For more information about Amazon GameLift, see  [Amazon GameLift](https://docs.aws.amazon.com/gamelift). For more
information about the Amazon GameLift Plugin for Unreal Engine,
see [Integrating Games with the Amazon GameLift Plugin for UnrealEngine]()
.

## Installing the Amazon GameLift Plugin for Unreal Engine

1. Download the Amazon GameLift Plugin for Unreal Engine. You can find the latest version on the **Amazon GameLift Plugin for
   UnrealEngine** GitHub repository. Under the [latest release](https://github.com)
2. Create a Plugins folder in the main project directory.\
**Note:** The folder name must be "plugins" or "Plugins".
3. Extract the downloaded folder from the .zip file and copy it to the Plugins folder. 
4. Launch Unreal Engine and select a project.
5. On the top menu bar, select **Edit**, and then choose **Plugins**.
6. Navigate to **Projects -> Other** and then enable Gamelift Plugin.
7. Once the plugin is loaded, **GameLiftPlugin** will be added as a new tab on the Unreal Editor Toolbar Menu.

## Setting up for Local Testing

1. Download testing tools according to [AWS documentation](https://docs.aws.amazon.com/gamelift/latest/developerguide/integration-testing-local.html#integration-testing-local-start)
2. Unzip the file downloaded
3. At the Unreal Editor Toolbar Menu navigate **Settings -> Project Settings**, select **GameLift Plugin** to configure the filepath of the `GameLiftLocal.jar`. 
4. If you haven't installed JRE, select **Download JRE** to download and install JRE from the official website.
5. Open **Run GameLift Local Game**
6. Specify **GameLift Local Port**.
7. Select Launch GameLift Local Server.\
   This will execute GameLift Local (via `java -jar <path_to_gamelift_local_jar> -p <port>`)
6. Set the path to your GameLift SDK integrated server executable
   * For game servers built against the Windows platform, this executable should be a `.exe` file
   * For game servers built against the Mac OS platform, this executable should be a `.app` file 
7. Select **Launch game server**
8. If the GameLift Server SDK is configured correctly in your server executable, you should see\
   `Healthcheck status: true` in GameLift Local terminal

## Deploying a Scenario

### First, configure the AWS Account credentials:

1. Go to the AWS Account Settings section following **Menu -> Edit -> Project Settings -> GameLift Plugin**, to setup a new Credentials Profile.
2. In the opened **AWS Credentials** section, create new credentials or select existing credentials.\
   **Note:** To manage your AWS credentials, follow the instructions described in the [AWS documentation](https://docs.aws.amazon.com/general/latest/gr/aws-sec-cred-types.html).

### Next, deploy a sample scenario:

**NOTE:** Only Windows server executables are supported in the plugin at the moment. On Mac, you'll need to build the game
server using Windows platform and use it for deployment, even though your local testing was done using game server built 
against Mac OS platform.

1. In the Plugin for Unreal Engine tab, select the **Deployment** tab.
2. In the Deployment window, select a scenario. The **Auth Only** scenario does not require a server executable and can
   deploy quickly. All other scenarios require a server path and server executable and can take about 30 minutes to
   deploy.
4. Specify a **Game Name**. It must be unique. It will be used as part of the AWS CloudFormation stack name when the
   scenario is deployed.
5. Select the **Build Folder Path**. The build folder path points to the folder containing the server
   executable and dependencies. For example, `c:/SampleGame/GameServer`. You will not be able to select a build folder
   path if it is not required by the chosen scenario.
6. Select the **Build File Path**. The build executable file path points to the game server executable.
   For example, `c:/SampleGame/GameServer/SampleGame.exe`. You will not be able to select a build executable file path
   if it is not required by the chosen scenario.
7. Select or left as default **Additional Server Resources Path**. A folder which containes additional files for GameLift server delivery. You will not be able to select a path if it is not required by the chosen scenario.
8. Select or left as default **Client Configuration Output Path**. A folder where the results of deployment will be stored. You will not be able to select a path if it is not required by the chosen scenario.
9. Select **Start Deployment** to initiate deployment of the scenario. The stack statuses and details will be displayed in the Current State section in the Deployment window. 

### How to set up the project development environment?

MS Windows OS is required. To build the Plugin, you need to install some dependencies. This will require administrator
rights on your machine:

* A supported Unreal Engine version
* Visual Studio 2019
* .NET 4.5 Developer Pack to build the Server SDK (NOTE: 4.5.1 or 4.5.2 does not work.) Due
  to [.NET 4.5 reaching end of life](https://dotnet.microsoft.com/download/dotnet-framework/net45). You can only install
  4.5 by following these steps:
    1. Open the Visual Studio Installer application. You should find your Visual Studio installation.
    2. Press Modify on your installation.
    3. Go to the Individual components tab.
    4. Check ".NET 4.5 Framework targeting pack", and press "Modify".
* .NET 4.7.1 Developer Pack to build AmazonGameLiftPlugin.Core. This can be downloaded
  at https://dotnet.microsoft.com/download/visual-studio-sdks, or as a part of MS Visual Studio.

## License