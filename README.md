<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>캐릭터 시트 메이커 Pro</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/fabric.js/5.3.0/fabric.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Noto Sans KR', 'Segoe UI', -apple-system, BlinkMacSystemFont, sans-serif;
            background: #1a1a2e;
            color: #333;
            overflow: hidden;
            height: 100vh;
        }

        .app-container {
            display: grid;
            grid-template-columns: 280px 1fr 280px;
            grid-template-rows: 60px 1fr 40px;
            height: 100vh;
            background: #16213e;
        }

        /* 헤더 */
        .header {
            grid-column: 1 / -1;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            display: flex;
            align-items: center;
            padding: 0 20px;
            box-shadow: 0 2px 10px rgba(0,0,0,0.2);
            z-index: 100;
        }

        .header h1 {
            color: white;
            font-size: 24px;
            font-weight: 600;
            margin-right: auto;
        }

        .header-buttons {
            display: flex;
            gap: 10px;
        }

        .header-btn {
            padding: 8px 16px;
            background: rgba(255,255,255,0.2);
            color: white;
            border: 1px solid rgba(255,255,255,0.3);
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.3s;
            font-size: 14px;
            font-family: 'Noto Sans KR', sans-serif;
        }

        .header-btn:hover {
            background: rgba(255,255,255,0.3);
            transform: translateY(-1px);
        }

        /* 좌측 툴바 */
        .left-toolbar {
            background: #1e2936;
            border-right: 1px solid #2a3f5f;
            overflow-y: auto;
            padding: 20px;
        }

        .tool-section {
            margin-bottom: 25px;
            background: #243447;
            border-radius: 8px;
            padding: 15px;
        }

        .tool-section h3 {
            color: #a8b2c7;
            font-size: 13px;
            font-weight: 600;
            margin-bottom: 12px;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .tool-btn {
            width: 100%;
            padding: 10px;
            margin-bottom: 8px;
            background: #2a3f5f;
            color: #fff;
            border: 1px solid #3a4f6f;
            border-radius: 6px;
            cursor: pointer;
            transition: all 0.3s;
            font-size: 14px;
            font-family: 'Noto Sans KR', sans-serif;
            display: flex;
            align-items: center;
            justify-content: center;
            gap: 8px;
        }

        .tool-btn:hover {
            background: #3a4f6f;
            transform: translateX(2px);
        }

        .tool-btn.active {
            background: #667eea;
            border-color: #667eea;
        }

        .tool-btn-group {
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 8px;
            margin-bottom: 10px;
        }

        .tool-btn-small {
            padding: 8px;
            font-size: 12px;
        }

        /* 우측 속성 패널 */
        .right-panel {
            background: #1e2936;
            border-left: 1px solid #2a3f5f;
            overflow-y: auto;
            padding: 20px;
        }

        .property-group {
            background: #243447;
            border-radius: 8px;
            padding: 15px;
            margin-bottom: 15px;
        }

        .property-group h4 {
            color: #a8b2c7;
            font-size: 12px;
            font-weight: 600;
            margin-bottom: 12px;
            text-transform: uppercase;
        }

        .property-row {
            display: flex;
            align-items: center;
            margin-bottom: 10px;
            gap: 10px;
        }

        .property-label {
            color: #8892a6;
            font-size: 13px;
            min-width: 80px;
        }

        .property-input {
            flex: 1;
            padding: 6px 10px;
            background: #1a2332;
            color: #fff;
            border: 1px solid #2a3f5f;
            border-radius: 4px;
            font-size: 13px;
            font-family: 'Noto Sans KR', sans-serif;
        }

        input[type="range"] {
            flex: 1;
            height: 6px;
            background: #2a3f5f;
            border-radius: 3px;
            outline: none;
            -webkit-appearance: none;
        }

        input[type="range"]::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 16px;
            height: 16px;
            background: #667eea;
            border-radius: 50%;
            cursor: pointer;
        }

        input[type="color"] {
            width: 50px;
            height: 35px;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }

        .range-value {
            color: #a8b2c7;
            font-size: 12px;
            min-width: 40px;
            text-align: right;
        }

        /* 메인 캔버스 영역 */
        .canvas-area {
            background: #0f1419;
            display: flex;
            justify-content: center;
            align-items: center;
            position: relative;
            overflow: hidden;
        }

        .canvas-wrapper {
            background: white;
            box-shadow: 0 10px 40px rgba(0,0,0,0.5);
            position: relative;
        }

        /* 하단 상태바 */
        .status-bar {
            grid-column: 1 / -1;
            background: #1a2332;
            border-top: 1px solid #2a3f5f;
            display: flex;
            align-items: center;
            padding: 0 20px;
            color: #8892a6;
            font-size: 12px;
        }

        .status-item {
            margin-right: 20px;
            display: flex;
            align-items: center;
            gap: 5px;
        }

        /* 파일 업로드 */
        input[type="file"] {
            display: none;
        }

        .file-label {
            display: block;
            width: 100%;
            padding: 10px;
            background: #4CAF50;
            color: white;
            border-radius: 6px;
            text-align: center;
            cursor: pointer;
            transition: all 0.3s;
            margin-bottom: 8px;
            font-family: 'Noto Sans KR', sans-serif;
        }

        .file-label:hover {
            background: #45a049;
        }

        /* 내보내기 모달 */
        .modal {
            display: none;
            position: fixed;
            z-index: 1000;
            left: 0;
            top: 0;
            width: 100%;
            height: 100%;
            background: rgba(0,0,0,0.7);
            justify-content: center;
            align-items: center;
        }

        .modal-content {
            background: #243447;
            padding: 30px;
            border-radius: 12px;
            box-shadow: 0 20px 60px rgba(0,0,0,0.5);
            min-width: 400px;
        }

        .modal-title {
            color: white;
            font-size: 20px;
            margin-bottom: 20px;
            font-family: 'Noto Sans KR', sans-serif;
        }

        .export-options {
            display: flex;
            gap: 10px;
            margin-bottom: 20px;
        }

        .export-btn {
            flex: 1;
            padding: 15px;
            background: #2a3f5f;
            color: white;
            border: 2px solid #3a4f6f;
            border-radius: 8px;
            cursor: pointer;
            transition: all 0.3s;
            text-align: center;
            font-family: 'Noto Sans KR', sans-serif;
        }

        .export-btn:hover {
            background: #667eea;
            border-color: #667eea;
            transform: translateY(-2px);
        }

        .close-modal {
            width: 100%;
            padding: 10px;
            background: #ff4757;
            color: white;
            border: none;
            border-radius: 6px;
            cursor: pointer;
            margin-top: 10px;
            font-family: 'Noto Sans KR', sans-serif;
        }

        /* 색상 팔레트 */
        .color-palette {
            display: grid;
            grid-template-columns: repeat(8, 1fr);
            gap: 5px;
            margin-top: 10px;
        }

        .color-preset {
            width: 25px;
            height: 25px;
            border-radius: 4px;
            cursor: pointer;
            border: 2px solid transparent;
            transition: all 0.2s;
        }

        .color-preset:hover {
            transform: scale(1.2);
            border-color: #667eea;
        }

        /* 도형 선택 */
        .shape-grid {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 8px;
        }

        .shape-btn {
            aspect-ratio: 1;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 20px;
        }

        /* 크롭 모드 */
        .crop-controls {
            position: absolute;
            bottom: 20px;
            left: 50%;
            transform: translateX(-50%);
            background: #243447;
            padding: 15px;
            border-radius: 8px;
            display: none;
            gap: 10px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.3);
            z-index: 200;
        }

        .crop-controls.active {
            display: flex;
        }

        .crop-btn {
            padding: 8px 16px;
            background: #667eea;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
            font-family: 'Noto Sans KR', sans-serif;
        }

        .crop-btn.cancel {
            background: #ff4757;
        }

        /* 텍스트 스타일 버튼 */
        .text-style-buttons {
            display: flex;
            gap: 5px;
            margin-bottom: 10px;
        }

        .style-btn {
            flex: 1;
            padding: 8px;
            background: #2a3f5f;
            color: white;
            border: 1px solid #3a4f6f;
            border-radius: 4px;
            cursor: pointer;
            font-size: 14px;
            transition: all 0.2s;
        }

        .style-btn.active {
            background: #667eea;
            border-color: #667eea;
        }

        .style-btn:hover {
            background: #3a4f6f;
        }

        /* 알림 메시지 */
        .notification {
            position: fixed;
            top: 80px;
            right: 20px;
            background: #4CAF50;
            color: white;
            padding: 15px 20px;
            border-radius: 8px;
            box-shadow: 0 4px 20px rgba(0,0,0,0.3);
            z-index: 1000;
            opacity: 0;
            transform: translateY(-20px);
            transition: all 0.3s;
            font-family: 'Noto Sans KR', sans-serif;
        }

        .notification.show {
            opacity: 1;
            transform: translateY(0);
        }
    </style>
</head>
<body>
    <div class="app-container">
        <!-- 헤더 -->
        <div class="header">
            <h1>🎨 캐릭터 시트 메이커 Pro</h1>
            <div class="header-buttons">
                <button class="header-btn" onclick="resetCanvas()">🔄 새로 만들기</button>
                <button class="header-btn" onclick="openExportModal()">💾 내보내기</button>
                <button class="header-btn" onclick="showHelp()">❓ 도움말</button>
            </div>
        </div>

        <!-- 좌측 툴바 -->
        <div class="left-toolbar">
            <div class="tool-section">
                <h3>📁 이미지</h3>
                <input type="file" id="imageUpload" accept="image/*" multiple>
                <label for="imageUpload" class="file-label">
                    이미지 업로드
                </label>
                <button class="tool-btn tool-btn-small" onclick="startCrop()">✂️ 자르기</button>
                <button class="tool-btn tool-btn-small" onclick="flipHorizontal()">↔️ 좌우반전</button>
                <button class="tool-btn tool-btn-small" onclick="flipVertical()">↕️ 상하반전</button>
            </div>

            <div class="tool-section">
                <h3>✏️ 그리기 도구</h3>
                <button class="tool-btn" id="selectMode" onclick="setMode('select')">👆 선택</button>
                <button class="tool-btn" id="drawMode" onclick="setMode('draw')">✏️ 펜</button>
                <div class="property-row">
                    <input type="range" id="brushSize" min="1" max="50" value="5">
                    <span class="range-value"><span id="brushSizeValue">5</span>px</span>
                </div>
            </div>

            <div class="tool-section">
                <h3>📝 텍스트</h3>
                <input type="text" id="textInput" class="property-input" placeholder="텍스트 입력">
                <div class="text-style-buttons">
                    <button class="style-btn" id="boldBtn" onclick="toggleBold()">B</button>
                    <button class="style-btn" id="italicBtn" onclick="toggleItalic()">I</button>
                    <button class="style-btn" id="underlineBtn" onclick="toggleUnderline()">U</button>
                </div>
                <button class="tool-btn" onclick="addText()">텍스트 추가</button>
            </div>

            <div class="tool-section">
                <h3>🔷 도형</h3>
                <div class="shape-grid">
                    <button class="tool-btn shape-btn" onclick="addShape('rect')">▭</button>
                    <button class="tool-btn shape-btn" onclick="addShape('circle')">○</button>
                    <button class="tool-btn shape-btn" onclick="addShape('triangle')">△</button>
                    <button class="tool-btn shape-btn" onclick="addShape('line')">╱</button>
                    <button class="tool-btn shape-btn" onclick="addShape('arrow')">→</button>
                    <button class="tool-btn shape-btn" onclick="addShape('star')">★</button>
                </div>
            </div>

            <div class="tool-section">
                <h3>⚙️ 편집</h3>
                <button class="tool-btn" onclick="bringToFront()">⬆️ 앞으로</button>
                <button class="tool-btn" onclick="sendToBack()">⬇️ 뒤로</button>
                <button class="tool-btn" onclick="duplicateObject()">📋 복제</button>
                <button class="tool-btn" onclick="deleteSelected()" style="background: #ff4757;">🗑️ 삭제</button>
            </div>
        </div>

        <!-- 캔버스 영역 -->
        <div class="canvas-area">
            <div class="canvas-wrapper">
                <canvas id="canvas"></canvas>
            </div>
            
            <!-- 크롭 컨트롤 -->
            <div class="crop-controls" id="cropControls">
                <button class="crop-btn" onclick="applyCrop()">✅ 자르기 적용</button>
                <button class="crop-btn cancel" onclick="cancelCrop()">❌ 취소</button>
            </div>
        </div>

        <!-- 우측 속성 패널 -->
        <div class="right-panel">
            <div class="property-group">
                <h4>캔버스 설정</h4>
                <div class="property-row">
                    <span class="property-label">너비</span>
                    <input type="number" id="canvasWidth" class="property-input" value="1200" min="100" max="3000">
                </div>
                <div class="property-row">
                    <span class="property-label">높이</span>
                    <input type="number" id="canvasHeight" class="property-input" value="800" min="100" max="3000">
                </div>
                <div class="property-row">
                    <span class="property-label">배경색</span>
                    <input type="color" id="bgColor" value="#e8f5e9">
                </div>
                <button class="tool-btn" onclick="applyCanvasSettings()">적용</button>
            </div>

            <div class="property-group">
                <h4>색상 설정</h4>
                <div class="property-row">
                    <span class="property-label">선 색상</span>
                    <input type="color" id="strokeColor" value="#000000">
                </div>
                <div class="property-row">
                    <span class="property-label">채우기</span>
                    <input type="color" id="fillColor" value="#ffffff">
                </div>
                <div class="color-palette" id="colorPalette"></div>
            </div>

            <div class="property-group">
                <h4>텍스트 속성</h4>
                <div class="property-row">
                    <span class="property-label">폰트</span>
                    <select id="fontFamily" class="property-input">
                        <option value="Noto Sans KR">Noto Sans KR</option>
                        <option value="Arial">Arial</option>
                        <option value="Helvetica">Helvetica</option>
                        <option value="Times New Roman">Times New Roman</option>
                        <option value="Georgia">Georgia</option>
                        <option value="Verdana">Verdana</option>
                        <option value="Comic Sans MS">Comic Sans MS</option>
                    </select>
                </div>
                <div class="property-row">
                    <span class="property-label">크기</span>
                    <input type="range" id="fontSize" min="10" max="120" value="30">
                    <span class="range-value"><span id="fontSizeValue">30</span></span>
                </div>
                <div class="property-row">
                    <span class="property-label">정렬</span>
                    <select id="textAlign" class="property-input">
                        <option value="left">왼쪽</option>
                        <option value="center">가운데</option>
                        <option value="right">오른쪽</option>
                    </select>
                </div>
            </div>

            <div class="property-group">
                <h4>객체 속성</h4>
                <div class="property-row">
                    <span class="property-label">투명도</span>
                    <input type="range" id="opacity" min="0" max="100" value="100">
                    <span class="range-value"><span id="opacityValue">100</span>%</span>
                </div>
                <div class="property-row">
                    <span class="property-label">회전</span>
                    <input type="range" id="rotation" min="0" max="360" value="0">
                    <span class="range-value"><span id="rotationValue">0</span>°</span>
                </div>
                <div class="property-row">
                    <span class="property-label">선 두께</span>
                    <input type="range" id="strokeWidth" min="0" max="20" value="2">
                    <span class="range-value"><span id="strokeWidthValue">2</span></span>
                </div>
            </div>

            <div class="property-group">
                <h4>필터 효과</h4>
                <button class="tool-btn tool-btn-small" onclick="applyFilter('grayscale')">흑백</button>
                <button class="tool-btn tool-btn-small" onclick="applyFilter('sepia')">세피아</button>
                <button class="tool-btn tool-btn-small" onclick="applyFilter('invert')">반전</button>
                <button class="tool-btn tool-btn-small" onclick="applyFilter('blur')">블러</button>
                <button class="tool-btn tool-btn-small" onclick="removeFilter()">필터 제거</button>
            </div>
        </div>

        <!-- 상태바 -->
        <div class="status-bar">
            <div class="status-item">
                <span>캔버스: </span>
                <span id="canvasSize">1200 x 800px</span>
            </div>
            <div class="status-item">
                <span>선택된 객체: </span>
                <span id="selectedObject">없음</span>
            </div>
            <div class="status-item">
                <span>객체 수: </span>
                <span id="objectCount">0</span>
            </div>
            <div class="status-item" style="margin-left: auto;">
                <span>줌: </span>
                <span id="zoomLevel">100%</span>
            </div>
        </div>
    </div>

    <!-- 내보내기 모달 -->
    <div id="exportModal" class="modal">
        <div class="modal-content">
            <h2 class="modal-title">📥 내보내기</h2>
            <div class="export-options">
                <div class="export-btn" onclick="exportImage('png')">
                    <div>📸 PNG</div>
                    <small style="opacity: 0.7; font-size: 11px;">투명 배경 지원</small>
                </div>
                <div class="export-btn" onclick="exportImage('jpg')">
                    <div>🖼️ JPG</div>
                    <small style="opacity: 0.7; font-size: 11px;">작은 파일 크기</small>
                </div>
                <div class="export-btn" onclick="exportImage('pdf')">
                    <div>📄 PDF</div>
                    <small style="opacity: 0.7; font-size: 11px;">문서 형식</small>
                </div>
            </div>
            <div class="property-row">
                <span class="property-label" style="color: #a8b2c7;">품질</span>
                <span style="color: #fff;">300 DPI (고품질)</span>
            </div>
            <button class="close-modal" onclick="closeExportModal()">닫기</button>
        </div>
    </div>

    <!-- 알림 메시지 -->
    <div id="notification" class="notification"></div>

    <script>
        // 캔버스 초기화
        const canvas = new fabric.Canvas('canvas', {
            width: 1200,
            height: 800,
            backgroundColor: '#e8f5e9'
        });

        let currentMode = 'select';
        let cropRect = null;
        let isCropping = false;
        let textBold = false;
        let textItalic = false;
        let textUnderline = false;
        let clipboard = null;
        let lastUploadedImage = null;

        // 알림 표시 함수
        function showNotification(message) {
            const notification = document.getElementById('notification');
            notification.textContent = message;
            notification.classList.add('show');
            setTimeout(() => {
                notification.classList.remove('show');
            }, 3000);
        }

        // 색상 팔레트 생성
        const colors = [
            '#000000', '#ffffff', '#ff0000', '#00ff00', '#0000ff', '#ffff00', '#ff00ff', '#00ffff',
            '#ff6b6b', '#4ecdc4', '#45b7d1', '#96ceb4', '#ffeaa7', '#dfe6e9', '#b2bec3', '#636e72',
            '#fab1a0', '#ff7675', '#fd79a8', '#e17055', '#fdcb6e', '#6c5ce7', '#a29bfe', '#74b9ff'
        ];

        colors.forEach(color => {
            const colorDiv = document.createElement('div');
            colorDiv.className = 'color-preset';
            colorDiv.style.backgroundColor = color;
            colorDiv.onclick = () => {
                document.getElementById('strokeColor').value = color;
                if (currentMode === 'draw') {
                    canvas.freeDrawingBrush.color = color;
                }
            };
            document.getElementById('colorPalette').appendChild(colorDiv);
        });

        // 이벤트 리스너
        document.getElementById('brushSize').addEventListener('input', function(e) {
            document.getElementById('brushSizeValue').textContent = e.target.value;
            if (canvas.freeDrawingBrush) {
                canvas.freeDrawingBrush.width = parseInt(e.target.value);
            }
        });

        document.getElementById('fontSize').addEventListener('input', function(e) {
            document.getElementById('fontSizeValue').textContent = e.target.value;
            updateSelectedText();
        });

        document.getElementById('opacity').addEventListener('input', function(e) {
            document.getElementById('opacityValue').textContent = e.target.value;
            updateSelectedObject();
        });

        document.getElementById('rotation').addEventListener('input', function(e) {
            document.getElementById('rotationValue').textContent = e.target.value;
            updateSelectedObject();
        });

        document.getElementById('strokeWidth').addEventListener('input', function(e) {
            document.getElementById('strokeWidthValue').textContent = e.target.value;
            updateSelectedObject();
        });

        // 모드 설정
        function setMode(mode) {
            currentMode = mode;
            
            // 모든 모드 버튼 비활성화
            document.querySelectorAll('.tool-btn').forEach(btn => btn.classList.remove('active'));
            
            // 현재 모드 버튼 활성화
            document.getElementById(mode + 'Mode')?.classList.add('active');
            
            // 그리기 모드 설정
            canvas.isDrawingMode = false;
            
            if (mode === 'select') {
                canvas.selection = true;
            } else if (mode === 'draw') {
                canvas.isDrawingMode = true;
                canvas.freeDrawingBrush = new fabric.PencilBrush(canvas);
                canvas.freeDrawingBrush.color = document.getElementById('strokeColor').value;
                canvas.freeDrawingBrush.width = parseInt(document.getElementById('brushSize').value);
            }
        }

        // 이미지 업로드
        document.getElementById('imageUpload').addEventListener('change', function(e) {
            const files = e.target.files;
            
            Array.from(files).forEach(file => {
                const reader = new FileReader();
                
                reader.onload = function(event) {
                    fabric.Image.fromURL(event.target.result, function(img) {
                        // 캔버스 크기에 맞게 조정
                        const maxWidth = canvas.width * 0.5;
                        const maxHeight = canvas.height * 0.5;
                        
                        if (img.width > maxWidth || img.height > maxHeight) {
                            const scale = Math.min(maxWidth / img.width, maxHeight / img.height);
                            img.scale(scale);
                        }
                        
                        img.set({
                            left: Math.random() * (canvas.width - 200),
                            top: Math.random() * (canvas.height - 200)
                        });
                        
                        canvas.add(img);
                        canvas.setActiveObject(img);
                        lastUploadedImage = img;
                        updateStatus();
                        showNotification('이미지가 업로드되었습니다. 자르기를 원하시면 자르기 버튼을 클릭하세요.');
                    });
                };
                
                reader.readAsDataURL(file);
            });
        });

        // 텍스트 스타일 토글
        function toggleBold() {
            textBold = !textBold;
            document.getElementById('boldBtn').classList.toggle('active', textBold);
            updateSelectedText();
        }

        function toggleItalic() {
            textItalic = !textItalic;
            document.getElementById('italicBtn').classList.toggle('active', textItalic);
            updateSelectedText();
        }

        function toggleUnderline() {
            textUnderline = !textUnderline;
            document.getElementById('underlineBtn').classList.toggle('active', textUnderline);
            updateSelectedText();
        }

        // 텍스트 추가
        function addText() {
            const text = document.getElementById('textInput').value || '텍스트';
            const textObj = new fabric.IText(text, {
                left: canvas.width / 2 - 50,
                top: canvas.height / 2 - 20,
                fontSize: parseInt(document.getElementById('fontSize').value),
                fill: document.getElementById('fillColor').value,
                fontFamily: document.getElementById('fontFamily').value,
                fontWeight: textBold ? 'bold' : 'normal',
                fontStyle: textItalic ? 'italic' : 'normal',
                underline: textUnderline,
                textAlign: document.getElementById('textAlign').value
            });
            
            canvas.add(textObj);
            canvas.setActiveObject(textObj);
            canvas.renderAll();
            document.getElementById('textInput').value = '';
            updateStatus();
        }

        // 도형 추가
        function addShape(type) {
            let shape;
            const strokeColor = document.getElementById('strokeColor').value;
            const fillColor = document.getElementById('fillColor').value;
            const strokeWidth = parseInt(document.getElementById('strokeWidth').value);
            
            switch(type) {
                case 'rect':
                    shape = new fabric.Rect({
                        width: 100,
                        height: 100,
                        fill: fillColor,
                        stroke: strokeColor,
                        strokeWidth: strokeWidth
                    });
                    break;
                case 'circle':
                    shape = new fabric.Circle({
                        radius: 50,
                        fill: fillColor,
                        stroke: strokeColor,
                        strokeWidth: strokeWidth
                    });
                    break;
                case 'triangle':
                    shape = new fabric.Triangle({
                        width: 100,
                        height: 100,
                        fill: fillColor,
                        stroke: strokeColor,
                        strokeWidth: strokeWidth
                    });
                    break;
                case 'line':
                    shape = new fabric.Line([0, 0, 100, 100], {
                        stroke: strokeColor,
                        strokeWidth: strokeWidth
                    });
                    break;
                case 'arrow':
                    const points = [0, 0, 100, 0];
                    const arrow = new fabric.Line(points, {
                        stroke: strokeColor,
                        strokeWidth: strokeWidth
                    });
                    const triangle = new fabric.Triangle({
                        width: 15,
                        height: 15,
                        fill: strokeColor,
                        left: 100,
                        top: -7.5,
                        angle: 90
                    });
                    shape = new fabric.Group([arrow, triangle]);
                    break;
                case 'star':
                    const starPoints = [];
                    const outerRadius = 50;
                    const innerRadius = 25;
                    const numPoints = 5;
                    
                    for (let i = 0; i < numPoints * 2; i++) {
                        const radius = i % 2 === 0 ? outerRadius : innerRadius;
                        const angle = (i * Math.PI) / numPoints - Math.PI / 2;
                        starPoints.push({
                            x: Math.cos(angle) * radius,
                            y: Math.sin(angle) * radius
                        });
                    }
                    
                    shape = new fabric.Polygon(starPoints, {
                        fill: fillColor,
                        stroke: strokeColor,
                        strokeWidth: strokeWidth
                    });
                    break;
            }
            
            if (shape) {
                shape.set({
                    left: canvas.width / 2 - 50,
                    top: canvas.height / 2 - 50
                });
                canvas.add(shape);
                canvas.setActiveObject(shape);
                canvas.renderAll();
                updateStatus();
            }
        }

        // 자르기 기능
        function startCrop() {
            const activeObject = canvas.getActiveObject();
            if (!activeObject || activeObject.type !== 'image') {
                alert('이미지를 먼저 선택해주세요!');
                return;
            }
            
            isCropping = true;
            document.getElementById('cropControls').classList.add('active');
            
            // 크롭 영역 생성
            cropRect = new fabric.Rect({
                left: activeObject.left,
                top: activeObject.top,
                width: activeObject.width * activeObject.scaleX * 0.8,
                height: activeObject.height * activeObject.scaleY * 0.8,
                fill: 'transparent',
                stroke: '#667eea',
                strokeWidth: 2,
                strokeDashArray: [5, 5],
                cornerColor: '#667eea',
                cornerSize: 10,
                transparentCorners: false,
                excludeFromExport: true
            });
            
            canvas.add(cropRect);
            canvas.setActiveObject(cropRect);
            canvas.renderAll();
        }

        function applyCrop() {
            if (!cropRect || !isCropping) return;
            
            const activeImage = canvas.getObjects('image').find(img => {
                const imgBounds = img.getBoundingRect();
                const cropBounds = cropRect.getBoundingRect();
                return imgBounds.left <= cropBounds.left + cropBounds.width &&
                       imgBounds.left + imgBounds.width >= cropBounds.left &&
                       imgBounds.top <= cropBounds.top + cropBounds.height &&
                       imgBounds.top + imgBounds.height >= cropBounds.top;
            });
            
            if (activeImage) {
                const cropBounds = cropRect.getBoundingRect();
                const imgBounds = activeImage.getBoundingRect();
                
                // 새로운 크롭된 이미지 생성
                const cropX = (cropBounds.left - imgBounds.left) / activeImage.scaleX;
                const cropY = (cropBounds.top - imgBounds.top) / activeImage.scaleY;
                const cropWidth = cropBounds.width / activeImage.scaleX;
                const cropHeight = cropBounds.height / activeImage.scaleY;
                
                activeImage.set({
                    cropX: cropX,
                    cropY: cropY,
                    width: cropWidth,
                    height: cropHeight
                });
                
                activeImage.setCoords();
            }
            
            canvas.remove(cropRect);
            cropRect = null;
            isCropping = false;
            document.getElementById('cropControls').classList.remove('active');
            canvas.renderAll();
            showNotification('이미지가 자르기되었습니다.');
        }

        function cancelCrop() {
            if (cropRect) {
                canvas.remove(cropRect);
                cropRect = null;
            }
            isCropping = false;
            document.getElementById('cropControls').classList.remove('active');
            canvas.renderAll();
        }

        // 반전 기능
        function flipHorizontal() {
            const activeObject = canvas.getActiveObject();
            if (activeObject) {
                activeObject.set('flipX', !activeObject.flipX);
                canvas.renderAll();
            }
        }

        function flipVertical() {
            const activeObject = canvas.getActiveObject();
            if (activeObject) {
                activeObject.set('flipY', !activeObject.flipY);
                canvas.renderAll();
            }
        }

        // 필터 적용
        function applyFilter(filterType) {
            const activeObject = canvas.getActiveObject();
            if (!activeObject || activeObject.type !== 'image') {
                alert('이미지를 선택해주세요!');
                return;
            }
            
            let filter;
            switch(filterType) {
                case 'grayscale':
                    filter = new fabric.Image.filters.Grayscale();
                    break;
                case 'sepia':
                    filter = new fabric.Image.filters.Sepia();
                    break;
                case 'invert':
                    filter = new fabric.Image.filters.Invert();
                    break;
                case 'blur':
                    filter = new fabric.Image.filters.Blur({ blur: 0.5 });
                    break;
            }
            
            if (filter) {
                activeObject.filters = [filter];
                activeObject.applyFilters();
                canvas.renderAll();
            }
        }

        function removeFilter() {
            const activeObject = canvas.getActiveObject();
            if (activeObject && activeObject.type === 'image') {
                activeObject.filters = [];
                activeObject.applyFilters();
                canvas.renderAll();
            }
        }

        // 객체 편집 기능
        function bringToFront() {
            const activeObject = canvas.getActiveObject();
            if (activeObject) {
                canvas.bringToFront(activeObject);
                canvas.renderAll();
            }
        }

        function sendToBack() {
            const activeObject = canvas.getActiveObject();
            if (activeObject) {
                canvas.sendToBack(activeObject);
                canvas.renderAll();
            }
        }

        function duplicateObject() {
            const activeObjects = canvas.getActiveObjects();
            if (activeObjects.length === 0) return;
            
            activeObjects.forEach(obj => {
                obj.clone(function(cloned) {
                    cloned.set({
                        left: cloned.left + 20,
                        top: cloned.top + 20
                    });
                    canvas.add(cloned);
                });
            });
            
            canvas.discardActiveObject();
            canvas.renderAll();
            updateStatus();
            showNotification('객체가 복제되었습니다.');
        }

        function deleteSelected() {
            const activeObjects = canvas.getActiveObjects();
            if (activeObjects.length > 0) {
                activeObjects.forEach(obj => {
                    canvas.remove(obj);
                });
                canvas.discardActiveObject();
                canvas.renderAll();
                updateStatus();
                showNotification(`${activeObjects.length}개의 객체가 삭제되었습니다.`);
            }
        }

        // 캔버스 설정
        function applyCanvasSettings() {
            const width = parseInt(document.getElementById('canvasWidth').value);
            const height = parseInt(document.getElementById('canvasHeight').value);
            const bgColor = document.getElementById('bgColor').value;
            
            canvas.setWidth(width);
            canvas.setHeight(height);
            canvas.backgroundColor = bgColor;
            canvas.renderAll();
            
            document.getElementById('canvasSize').textContent = `${width} x ${height}px`;
            showNotification('캔버스 설정이 적용되었습니다.');
        }

        // 선택된 객체 업데이트
        function updateSelectedObject() {
            const activeObject = canvas.getActiveObject();
            if (!activeObject) return;
            
            const opacity = parseInt(document.getElementById('opacity').value) / 100;
            const rotation = parseInt(document.getElementById('rotation').value);
            const strokeWidth = parseInt(document.getElementById('strokeWidth').value);
            
            activeObject.set({
                opacity: opacity,
                angle: rotation,
                strokeWidth: strokeWidth
            });
            
            canvas.renderAll();
        }

        function updateSelectedText() {
            const activeObject = canvas.getActiveObject();
            if (!activeObject || activeObject.type !== 'i-text') return;
            
            activeObject.set({
                fontSize: parseInt(document.getElementById('fontSize').value),
                fontFamily: document.getElementById('fontFamily').value,
                fontWeight: textBold ? 'bold' : 'normal',
                fontStyle: textItalic ? 'italic' : 'normal',
                underline: textUnderline,
                fill: document.getElementById('fillColor').value,
                textAlign: document.getElementById('textAlign').value
            });
            
            canvas.renderAll();
        }

        // 내보내기 기능
        function openExportModal() {
            document.getElementById('exportModal').style.display = 'flex';
        }

        function closeExportModal() {
            document.getElementById('exportModal').style.display = 'none';
        }

        function exportImage(format) {
            // 크롭 사각형이 있으면 제거
            if (cropRect) {
                canvas.remove(cropRect);
            }
            
            const multiplier = 300 / 96; // 300 DPI
            
            try {
                if (format === 'pdf') {
                    // PDF 내보내기
                    const { jsPDF } = window.jspdf;
                    const imgWidth = canvas.width;
                    const imgHeight = canvas.height;
                    
                    // PDF 페이지 크기 계산 (mm 단위)
                    const pdfWidth = imgWidth * 0.264583; // px to mm
                    const pdfHeight = imgHeight * 0.264583;
                    
                    const pdf = new jsPDF({
                        orientation: imgWidth > imgHeight ? 'landscape' : 'portrait',
                        unit: 'mm',
                        format: [pdfWidth, pdfHeight]
                    });
                    
                    const dataURL = canvas.toDataURL({
                        format: 'png',
                        quality: 1.0,
                        multiplier: multiplier
                    });
                    
                    pdf.addImage(dataURL, 'PNG', 0, 0, pdfWidth, pdfHeight);
                    pdf.save('character-sheet.pdf');
                    showNotification('PDF 파일이 다운로드되었습니다.');
                } else {
                    // PNG/JPG 내보내기
                    canvas.toBlob(function(blob) {
                        const link = document.createElement('a');
                        link.download = `character-sheet.${format}`;
                        link.href = URL.createObjectURL(blob);
                        link.click();
                        URL.revokeObjectURL(link.href);
                        showNotification(`${format.toUpperCase()} 파일이 다운로드되었습니다.`);
                    }, `image/${format === 'png' ? 'png' : 'jpeg'}`, 1.0);
                }
            } catch (error) {
                console.error('Export error:', error);
                alert('내보내기 중 오류가 발생했습니다. 다시 시도해주세요.');
            }
            
            closeExportModal();
        }

        // 캔버스 초기화
        function resetCanvas() {
            if (confirm('모든 내용이 삭제됩니다. 계속하시겠습니까?')) {
                canvas.clear();
                canvas.backgroundColor = '#e8f5e9';
                canvas.renderAll();
                updateStatus();
                showNotification('캔버스가 초기화되었습니다.');
            }
        }

        // 도움말
        function showHelp() {
            alert(`🎨 캐릭터 시트 메이커 Pro 사용법

📁 이미지: 여러 이미지 업로드, 자르기, 반전 가능
✏️ 그리기: 펜으로 자유롭게 그리기
📝 텍스트: 굵게(B), 기울임(I), 밑줄(U) 스타일 적용
🔷 도형: 사각형, 원, 삼각형, 선, 화살표, 별 추가
⚙️ 편집: 객체 순서 변경, 복제, 삭제
🎨 필터: 흑백, 세피아, 반전, 블러 효과

단축키:
• Delete: 선택 객체 삭제 (다중 선택 가능)
• Ctrl+C: 복사
• Ctrl+V: 붙여넣기 (이미지도 가능)
• Ctrl+A: 전체 선택

💾 내보내기: PNG, JPG, PDF 형식 (300 DPI)`);
        }

        // 상태 업데이트
        function updateStatus() {
            const objects = canvas.getObjects().filter(obj => !obj.excludeFromExport);
            document.getElementById('objectCount').textContent = objects.length;
            
            const activeObjects = canvas.getActiveObjects();
            if (activeObjects.length > 0) {
                if (activeObjects.length === 1) {
                    let type = activeObjects[0].type;
                    if (type === 'i-text') type = '텍스트';
                    else if (type === 'image') type = '이미지';
                    else if (type === 'rect') type = '사각형';
                    else if (type === 'circle') type = '원';
                    else if (type === 'triangle') type = '삼각형';
                    else if (type === 'group') type = '그룹';
                    else if (type === 'path') type = '그리기';
                    document.getElementById('selectedObject').textContent = type;
                } else {
                    document.getElementById('selectedObject').textContent = `${activeObjects.length}개 선택됨`;
                }
            } else {
                document.getElementById('selectedObject').textContent = '없음';
            }
        }

        // 캔버스 이벤트 리스너
        canvas.on('selection:created', updateStatus);
        canvas.on('selection:updated', updateStatus);
        canvas.on('selection:cleared', updateStatus);
        canvas.on('object:added', updateStatus);
        canvas.on('object:removed', updateStatus);

        // 클립보드에서 이미지 붙여넣기
        document.addEventListener('paste', function(e) {
            const items = e.clipboardData.items;
            
            for (let i = 0; i < items.length; i++) {
                if (items[i].type.indexOf('image') !== -1) {
                    const blob = items[i].getAsFile();
                    const reader = new FileReader();
                    
                    reader.onload = function(event) {
                        fabric.Image.fromURL(event.target.result, function(img) {
                            const maxWidth = canvas.width * 0.5;
                            const maxHeight = canvas.height * 0.5;
                            
                            if (img.width > maxWidth || img.height > maxHeight) {
                                const scale = Math.min(maxWidth / img.width, maxHeight / img.height);
                                img.scale(scale);
                            }
                            
                            img.set({
                                left: canvas.width / 2 - img.getScaledWidth() / 2,
                                top: canvas.height / 2 - img.getScaledHeight() / 2
                            });
                            
                            canvas.add(img);
                            canvas.setActiveObject(img);
                            canvas.renderAll();
                            updateStatus();
                            showNotification('이미지가 붙여넣기되었습니다.');
                        });
                    };
                    
                    reader.readAsDataURL(blob);
                    e.preventDefault();
                    return;
                }
            }
        });

        // 키보드 단축키
        document.addEventListener('keydown', function(e) {
            // Delete 키 - 다중 선택 삭제
            if (e.key === 'Delete' || e.key === 'Backspace') {
                if (!document.activeElement.matches('input, select, textarea')) {
                    deleteSelected();
                    e.preventDefault();
                }
            }
            
            // Ctrl+C (복사)
            if (e.ctrlKey && e.key === 'c') {
                const activeObjects = canvas.getActiveObjects();
                if (activeObjects.length > 0) {
                    canvas.getActiveObject().clone(function(cloned) {
                        clipboard = cloned;
                    });
                    showNotification('객체가 복사되었습니다.');
                }
            }
            
            // Ctrl+V (붙여넣기)
            if (e.ctrlKey && e.key === 'v') {
                if (clipboard) {
                    clipboard.clone(function(clonedObj) {
                        canvas.discardActiveObject();
                        clonedObj.set({
                            left: clonedObj.left + 20,
                            top: clonedObj.top + 20,
                            evented: true,
                        });
                        
                        if (clonedObj.type === 'activeSelection') {
                            clonedObj.canvas = canvas;
                            clonedObj.forEachObject(function(obj) {
                                canvas.add(obj);
                            });
                            clonedObj.setCoords();
                        } else {
                            canvas.add(clonedObj);
                        }
                        
                        clipboard.top += 20;
                        clipboard.left += 20;
                        canvas.setActiveObject(clonedObj);
                        canvas.requestRenderAll();
                        showNotification('객체가 붙여넣기되었습니다.');
                    });
                }
            }
            
            // Ctrl+A (전체 선택)
            if (e.ctrlKey && e.key === 'a') {
                if (!document.activeElement.matches('input, select, textarea')) {
                    e.preventDefault();
                    canvas.discardActiveObject();
                    const allObjects = canvas.getObjects();
                    if (allObjects.length > 0) {
                        const selection = new fabric.ActiveSelection(allObjects, { canvas: canvas });
                        canvas.setActiveObject(selection);
                        canvas.renderAll();
                        updateStatus();
                    }
                }
            }
        });

        // 초기 상태 설정
        updateStatus();
        setMode('select');
        
        // 마우스 휠로 줌
        canvas.on('mouse:wheel', function(opt) {
            const delta = opt.e.deltaY;
            let zoom = canvas.getZoom();
            zoom *= 0.999 ** delta;
            if (zoom > 5) zoom = 5;
            if (zoom < 0.1) zoom = 0.1;
            canvas.setZoom(zoom);
            document.getElementById('zoomLevel').textContent = Math.round(zoom * 100) + '%';
            opt.e.preventDefault();
            opt.e.stopPropagation();
        });

        // 색상 변경 이벤트
        document.getElementById('strokeColor').addEventListener('change', function(e) {
            if (canvas.freeDrawingBrush) {
                canvas.freeDrawingBrush.color = e.target.value;
            }
            const activeObject = canvas.getActiveObject();
            if (activeObject && activeObject.stroke !== undefined) {
                activeObject.set('stroke', e.target.value);
                canvas.renderAll();
            }
        });

        document.getElementById('fillColor').addEventListener('change', function(e) {
            const activeObject = canvas.getActiveObject();
            if (activeObject && activeObject.fill !== undefined && activeObject.type !== 'path') {
                activeObject.set('fill', e.target.value);
                canvas.renderAll();
            }
        });

        // 텍스트 속성 변경 이벤트
        document.getElementById('fontFamily').addEventListener('change', updateSelectedText);
        document.getElementById('textAlign').addEventListener('change', updateSelectedText);

        // 초기 캔버스 크기 표시
        document.getElementById('canvasSize').textContent = '1200 x 800px';
        
        // 페이지 로드 시 환영 메시지
        window.addEventListener('load', function() {
            showNotification('캐릭터 시트 메이커 Pro에 오신 것을 환영합니다!');
        });
    </script>
</body>
</html>
