# This file contains the fastlane.tools configuration
# You can find the documentation at https://docs.fastlane.tools
#
# For a list of all available actions, check out
#
#     https://docs.fastlane.tools/actions
#
# For a list of all available plugins, check out
#
#     https://docs.fastlane.tools/plugins/available-plugins
#

# Uncomment the line if you want fastlane to automatically update itself
# update_fastlane

# WORKAROUND for failed uploads to Test Flight
# https://github.com/fastlane/fastlane/issues/21125
require 'spaceship'
Spaceship::ConnectAPI::App.const_set('ESSENTIAL_INCLUDES', 'appStoreVersions')

default_platform(:ios)

platform :ios do

  before_all do
    app_store_connect_api_key(
      key_id: "G9Q4HJ8G7C",
      issuer_id: "69a6de86-de62-47e3-e053-5b8c7c11a4d1",
      key_content: "MIGTAgEAMBMGByqGSM49AgEGCCqGSM49AwEHBHkwdwIBAQQgHn4wtSNdqMt3b8z2
4unyAna/w33L4eTbE/9XvwuBSRagCgYIKoZIzj0DAQehRANCAAT2en9OidGBrwyh
KrGD+abQqkQrSBdbISPYUNKPviAcyXrt2W3XVr6/Bdsu2tfCQFLZZOMMY8Gbv8Fk
PCQud2Pe",
      is_key_content_base64: true
    )
  end

  lane :dev_build do |options|
    desc "DEV build"
    get_certificates
    get_provisioning_profile(adhoc: "true")
    update_code_signing_settings(
      use_automatic_signing: false,
      path: "Unity-iPhone.xcodeproj",
      targets: "Unity-iPhone",
      code_sign_identity: "Apple Distribution: Gamesture Sp. z o.o.",
      profile_name: options[:provisioning_profile_name]
    )
    
    build_app(
      codesigning_identity: "Apple Distribution: Gamesture Sp. z o.o.",
      skip_profile_detection: true,
      export_method: "ad-hoc",
      clean: true,
      scheme: "Unity-iPhone",
      workspace: "Unity-iPhone.xcworkspace",
      output_directory: options[:build_output_dir], 
      export_options: {
        method: "ad-hoc",
        provisioningProfiles: { 
          options[:game_bundle_name] => options[:provisioning_profile_name]
        },
        manageAppVersionAndBuildNumber: false
      })
  end

  lane :prod_build do |options|
    desc "PROD build"
    get_certificates
    get_provisioning_profile
    update_code_signing_settings(
      use_automatic_signing: false,
      path: "Unity-iPhone.xcodeproj",
      targets: "Unity-iPhone",
      code_sign_identity: "Apple Distribution: Gamesture Sp. z o.o.",
      profile_name: options[:provisioning_profile_name]
    )
    
    build_app(
      codesigning_identity: "Apple Distribution: Gamesture Sp. z o.o.",
      skip_profile_detection: true,
      include_symbols: true,
      export_method: "app-store",
      clean: true,
      scheme: "Unity-iPhone",
      workspace: "Unity-iPhone.xcworkspace",
      output_directory: options[:build_output_dir], 
      export_options: {
        method: "app-store",
        provisioningProfiles: { 
          options[:game_bundle_name] => options[:provisioning_profile_name]
        },
        manageAppVersionAndBuildNumber: false
      })
  end

  lane :upload_to_apple do |options|
    desc "uploading to test flight"
    upload_to_testflight(
      skip_submission: false,
      skip_waiting_for_build_processing: true,
      ipa: options[:ipa_file])
  end
  
end

platform :android do
  
  lane :upload_to_google do |options|
    desc "uploading to google play internal"

    shrink_symbols(options)

    upload_to_play_store(
      root_url: "https://androidpublisher.googleapis.com/",
      track: "internal",
      skip_upload_apk: true,
      aab: options[:aab_file],
      mapping: options[:shrinked_file],
      package_name: options[:game_bundle_name],
      version_name: options[:game_version],
      timeout: 1800)
  end

  lane :shrink_symbols do |options|
    desc "shrinking android symbols file"

    sh("rm", "-rf", options[:shrinked_file])

    Dir.chdir("..") do
      sh 'rm -rf temp'
      sh 'mkdir temp'
      
      Dir.chdir("temp") do
        sh("unzip", options[:symbols_file])
        sh('find . -name "*.dbg.so" -exec rm "{}" \;')
        sh('find . -name "*.sym.so" | sed -e "p;s/\.sym\./\./" | xargs -n2 mv')
        sh("zip",  "-9",  "-r", options[:shrinked_file], ".")
      end

      sh 'rm -rf temp'

    end

  end

end
