
1. 260204
	1. MainView에서 각 채널 Show 버튼 클릭 시 View의 확대배율 변화 없이 해당 위치에서 CH만 바뀌게끔
		* 프롬프트 : 
		  LeDos_SVIView.cpp 파일 내 IDC_RED_IMG_SHOW, IDC_GREEN_IMG_SHOW, IDC_BLUE_IMG_SHOW 버튼의 클릭 이벤트 핸들러(ClickRedImgShow,
		  ClickGreenImgShow, ClickBlueImgShow 함수)를 다음과 같이 수정해주세요:
		
		
		   1. 메인 이미지 뷰(`IDC_IMAGE_VIEW_AUTO`)의 채널 변경:
		       * 각 버튼 클릭 시, m_CtrlImageMapView (메인 이미지 뷰)에 해당 채널(Red, Green, Blue)의 전체 이미지를 로드해야 합니다.
		       * 이때 m_CtrlImageMapView의 현재 확대/축소 배율과 패닝 위치는 반드시 그대로 유지되어야 합니다.
		       * 이 동작은 'View ALL' 체크박스의 활성화 여부와 관계없이 일관되게 적용되어야 합니다.
		
		
		   2. 디펙트 이미지 뷰(`IDC_DEFECT_IMAGE_VIEW`)는 영향을 받지 않음:
		       * m_CtrlImageDefectView (디펙트 이미지 뷰)는 이 버튼들의 클릭에 의해 어떠한 변경도 일어나지 않고 이전 상태를 유지해야 합니다.
		         즉, 이 버튼들은 m_CtrlImageDefectView를 건드리지 않아야 합니다.
		
		
		   3. 내부 플래그 설정:
		       * m_bImageViewDefect 플래그는 FALSE로 설정해야 합니다. 이는 메인 맵 뷰가 조작되는 상황임을 나타내고, 디펙트 뷰의 특정 디펙트
		         표시와는 관련이 없음을 의미합니다.
		
		
		  이 프롬프트는 핵심적인 요구사항을 모두 포함하고 있으며, 제가 수행해야 할 정확한 동작과 영향을 주거나 주지 말아야 할 컴포넌트를
		  명확히 명시하고 있습니다. 앞으로 이 형식으로 요청해주시면 됩니다!
		  
	1. InspAA의 AA ROI내의 평균 밝기를 OneChipLog에 표현하도록 
	   * 프롬프트 :
		  