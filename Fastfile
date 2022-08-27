default_platform(:ios)

platform :ios do

  desc "Connect App Store connect by API"
  lane :connectByAPI do
    app_store_connect_api_key(
      key_id: ENV['API_KEY_ID'],
      issuer_id: ENV['API_KEY_ISSUER_ID'],
      key_filepath: ENV['API_KEY_PATH']
    )
  end

  lane :createCertAndProfile do
    connectByAPI

    get_certificates(
      team_id: ENV['TEAM_ID'],
      output_path: "provision",
      keychain_password: ""
    )
    
    get_provisioning_profile(
      filename: "#{profileName}.mobileprovision",
      app_identifier:ENV['BUNDLE_ID'],
      output_path:"provision",
      cert_id: lane_context[SharedValues::CERT_CERTIFICATE_ID]
    )
  end

  desc "Build and archive APP"
  lane :archive do 
    update_code_signing_settings(
      path: "#{ENV['SCHEME']}.xcodeproj",
      team_id:ENV["TEAM_ID"],
      targets: ENV["SCHEME"],
      code_sign_identity: "Apple Distribution",
      profile_name: "#{profileName}",
      bundle_identifier: ENV["BUNDLE_ID"]
    )

    build_app(
      workspace: "#{ENV['SCHEME']}.xcworkspace",
      scheme: ENV["SCHEME"],
      export_method: "app-store",
      clean: true,
      output_directory: 'build',
      output_name: "#{fileName}",
      xcargs: 'DEBUG_INFORMATION_FORMAT=dwarf-with-dsym'
    )
  end

  lane :upload do 
    connectByAPI

    pilot(
      ipa: "build/#{fileName}",
      skip_waiting_for_build_processing: true
    )
  end

  lane :submit_pure do
    connectByAPI

    deliver(
      app_identifier: ENV['BUNDLE_ID'],
      submit_for_review: true, #用來送審
      automatic_release: true, #審過自動開放
      force: true, 
      skip_metadata: false, #不更新文案
      skip_screenshots: true, #不上傳截圖
      ipa: "build/#{fileName}",
      run_precheck_before_submit: false, #不預先檢查
      reject_if_possible: true, #如果有已經送審中的 ipa，就取消重送
      submission_information: {
	add_id_info_uses_idfa: false,
	export_compliance_uses_encryption: false
      }, #隱私權設定
      languages: ['en-US']
    )
  end

  private_lane :profileName do
    "#{ENV['BUNDLE_ID']} AppStore"
  end
  private_lane :fileName do
    versionNumber = get_version_number(target: ENV["SCHEME"])
    "#{ENV["SCHEME"]}_#{versionNumber}.ipa"
  end
end
