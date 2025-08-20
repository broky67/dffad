<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">
	<PropertyGroup>
		<TargetFramework>net472</TargetFramework>
		<ProjectTypeGuids>{60dc8134-eba5-43b8-bcc9-bb4bc16c2548};{FAE04EC0-301F-11D3-BF4B-00C04F79EFBC}</ProjectTypeGuids>
		<GenerateAssemblyInfo>false</GenerateAssemblyInfo>
		<OutputPath>bin\$(Configuration)\</OutputPath>
		<UseWPF>true</UseWPF>
	</PropertyGroup>
	<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
		<DebugType>full</DebugType>
	</PropertyGroup>
	<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Release|AnyCPU' ">
		<DebugType>pdbonly</DebugType>
	</PropertyGroup>
	<ItemGroup>
		<PackageReference Include="CommonServiceLocator" Version="1.3" />
		<PackageReference Include="Ekra.libiec61850.NET" Version="1.4.0" />
		<PackageReference Include="Ekra.Ookii.Dialogs" Version="1.0.1" />
		<PackageReference Include="Fluent.Ribbon" Version="3.6.1.236" />
		<PackageReference Include="Ekra.Prism.Contract" Version="4.1.1.0" />
		<PackageReference Include="Ekra.Extended.Wpf.Toolkit" Version="3.6.3" />
	</ItemGroup>
	<ItemGroup>
		<Reference Include="System.ComponentModel.Composition" />
	</ItemGroup>
	<ItemGroup>
		<Compile Update="Client61850.cs">
			<SubType>Component</SubType>
		</Compile>
		<Compile Update="Properties\Settings.Designer.cs">
		  <DesignTimeSharedInput>True</DesignTimeSharedInput>
		  <AutoGen>True</AutoGen>
		  <DependentUpon>Settings.settings</DependentUpon>
		</Compile>
	</ItemGroup>
	<ItemGroup>
		<ProjectReference Include="..\Framework\Catel.Core\Catel.Core.csproj" />
		<ProjectReference Include="..\Framework\Catel.MVVM\Catel.MVVM.csproj" />
		<ProjectReference Include="..\Pilot.Shell.Infrastructure\Pilot.Shell.Infrastructure.csproj" />
		<ProjectReference Include="..\kivi.Common\Pilot.Controls.Interactivity.csproj" />
		<ProjectReference Include="..\kivi.Controls\Pilot.Controls.csproj" />
		<ProjectReference Include="..\Pilot.Plugins\Pilot.Plugins.csproj" />
		<ProjectReference Include="..\SCLExpress.Core\SCLExpress.Core.csproj" />
	</ItemGroup>
	<ItemGroup>
		<Folder Include="Resources\" />
	</ItemGroup>
	<ItemGroup>
	  <None Update="Properties\Settings.settings">
	    <Generator>SettingsSingleFileGenerator</Generator>
	    <LastGenOutput>Settings.Designer.cs</LastGenOutput>
	  </None>
	</ItemGroup>
	<Target Name="PreBuild" BeforeTargets="PreBuildEvent">
		<Exec Command="$(SolutionDir)get-svn-revision.bat $(ProjectDir)" />
	</Target>
	<Target Name="PostBuild" AfterTargets="PostBuildEvent">
		<Exec Command="xcopy &quot;$(TargetDir)*.*&quot; &quot;$(SolutionDir)$(OutDir)&quot; /Y /S /D" />
	</Target>
</Project>


Microsoft.Practices.Prism.Modularity.ModuleTypeLoadingException: 'Failed to load type for module SearchEngineModule. 

If this error occurred when using MEF in a Silverlight application, please ensure that the CopyLocal property of the reference to the MefExtensions assembly is set to true in the main application/shell and false in all other assemblies. 

Error was: Unable to locate the module with type 'Module.SearchEngine.SearchEngineModule, Module.SearchEngine, Version=0.0.0.0, Culture=neutral, PublicKeyToken=null' among the exported modules. Make sure the module name in the module catalog matches that specified on ModuleExportAttribute for the module type..'

This exception was originally thrown at this call stack:
    Microsoft.Practices.Prism.Modularity.ModuleInitializer.HandleModuleInitializationError(Microsoft.Practices.Prism.Modularity.ModuleInfo, string, System.Exception)
    Microsoft.Practices.Prism.Modularity.ModuleInitializer.Initialize(Microsoft.Practices.Prism.Modularity.ModuleInfo)
    Microsoft.Practices.Prism.Modularity.ModuleManager.LoadModulesThatAreReadyForLoad()
    Microsoft.Practices.Prism.Modularity.ModuleManager.IModuleTypeLoader_LoadModuleCompleted(object, Microsoft.Practices.Prism.Modularity.LoadModuleCompletedEventArgs)
    Microsoft.Practices.Prism.MefExtensions.Modularity.MefFileModuleTypeLoader.RaiseLoadModuleCompleted(Microsoft.Practices.Prism.Modularity.LoadModuleCompletedEventArgs)
    Microsoft.Practices.Prism.MefExtensions.Modularity.MefFileModuleTypeLoader.LoadModuleType(Microsoft.Practices.Prism.Modularity.ModuleInfo)
