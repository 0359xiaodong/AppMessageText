AppMessageText
==============

Android角标-反编译QQ-apk所得, 为系统广播/Launcher提供(未深入研究)



  
    Intent intent = new Intent(
				"android.intent.action.APPLICATION_MESSAGE_UPDATE");
		// replace the extra value with your_package_name/your_main_activity_name
		intent.putExtra(
				"android.intent.extra.update_application_component_name",
				"android.intclub.net./.MainActivity");
		// replase the extra value with the text your want to be displayed
		intent.putExtra("android.intent.extra.update_application_message_text",
				"999");
		sendBroadcast(intent);
