<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CORTEX V15 // HIVE OVERLORD 2.0</title>
    
    <script src="https://cdn.jsdelivr.net/npm/hls.js@1.4.0/dist/hls.min.js"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css" rel="stylesheet">
    <link href="https://fonts.googleapis.com/css2?family=Orbitron:wght@400;700;900&family=Share+Tech+Mono&display=swap" rel="stylesheet">

    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-app.js"></script>
    <script src="https://www.gstatic.com/firebasejs/8.10.1/firebase-database.js"></script>

    <style>
        :root {
            --cyan: #00f3ff; 
            --cyan-glow: rgba(0, 243, 255, 0.4);
            --green: #0aff00; 
            --red: #ff0055; 
            --amber: #ffae00;
            --purple: #d000ff; 
            --gold: #ffd700;
            --bg-dark: #050505; 
            --panel-glass: rgba(10, 15, 20, 0.75);
            --font-ui: 'Orbitron', sans-serif; 
            --font-mono: 'Share Tech Mono', monospace;
        }

        * { box-sizing: border-box; }

        body {
            margin: 0; background: var(--bg-dark); color: #fff; height: 100vh;
            display: flex; flex-direction: column; overflow: hidden; 
            font-family: var(--font-mono);
            background-image: linear-gradient(rgba(0, 243, 255, 0.03) 1px, transparent 1px), linear-gradient(90deg, rgba(0, 243, 255, 0.03) 1px, transparent 1px);
            background-size: 30px 30px;
        }

        /* --- UI HEADER --- */
        header {
            display: flex; justify-content: space-between; align-items: center;
            padding: 12px 25px; border-bottom: 1px solid var(--cyan);
            background: rgba(0, 0, 0, 0.9); box-shadow: 0 0 20px var(--cyan-glow); z-index: 20;
        }

        .ai-brain { display: flex; align-items: center; gap: 15px; }
        .pulse { 
            width: 14px; height: 14px; background: var(--green); border-radius: 50%; 
            box-shadow: 0 0 15px var(--green); animation: pulse 2s infinite ease-in-out; 
        }
        @keyframes pulse { 0%, 100% { transform: scale(0.9); opacity: 0.7; } 50% { transform: scale(1.1); opacity: 1; } }

        .ai-status { display: flex; align-items: center; gap: 15px; }

        select, .btn-admin {
            background: rgba(0, 243, 255, 0.1); border: 1px solid var(--cyan); color: var(--cyan);
            padding: 6px 12px; font-family: var(--font-ui); font-weight: bold;
            outline: none; cursor: pointer; border-radius: 4px; transition: all 0.3s ease;
        }
        select:hover, .btn-admin:hover { background: var(--cyan); color: #000; }
        .btn-admin { border-color: var(--amber); color: var(--amber); background: rgba(255, 174, 0, 0.1); }
        .btn-admin:hover { background: var(--amber); color: #000; }

        /* --- LAYOUT --- */
        .main { display: flex; flex: 1; height: calc(100vh - 60px); position: relative; }

        .player-box { 
            flex: 1; background: #000; position: relative; display: flex;
            flex-direction: column; overflow: hidden; transition: all 0.3s ease;
            width: 100%;
        }
        
        .signal-glow {
            box-shadow: inset 0 0 40px var(--cyan), 0 0 30px var(--cyan) !important;
            border: 2px solid var(--cyan); animation: av1-pulse 2s infinite alternate;
        }
        @keyframes av1-pulse {
            0% { box-shadow: inset 0 0 20px var(--cyan), 0 0 15px var(--cyan); }
            100% { box-shadow: inset 0 0 60px var(--cyan), 0 0 40px var(--cyan); }
        }

        video { width: 100%; height: 100%; object-fit: contain; transition: filter 0.3s ease; }

        /* --- BOTON MAX FLOTANTE --- */
        #floating-max-btn {
            position: absolute; top: 25px; right: 25px; background: rgba(0, 243, 255, 0.15);
            border: 2px solid var(--cyan); color: var(--cyan); font-family: var(--font-ui); font-weight: 900;
            font-size: 1.1rem; padding: 8px 15px; border-radius: 8px; cursor: pointer; z-index: 50;
            backdrop-filter: blur(5px); box-shadow: 0 0 15px var(--cyan-glow), inset 0 0 10px rgba(0, 243, 255, 0.2);
            transition: opacity 0.5s ease, transform 0.3s ease, background 0.3s, box-shadow 0.3s;
            text-shadow: 0 0 5px var(--cyan);
        }
        #floating-max-btn:hover {
            background: var(--cyan); color: #000; box-shadow: 0 0 25px var(--cyan), inset 0 0 15px rgba(255, 255, 255, 0.5);
            transform: scale(1.05); text-shadow: none;
        }
        #floating-max-btn:active { transform: scale(0.95); }
        .hidden-max-btn { opacity: 0 !important; pointer-events: none !important; transform: translateY(-20px) !important; }

        /* --- HUD Y PANELES FLOTANTES --- */
        .osd {
            position: absolute; top: 25px; left: 25px; padding: 15px 25px;
            background: rgba(5, 5, 5, 0.65); border-left: 4px solid var(--cyan);
            backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px);
            border-radius: 0 8px 8px 0; box-shadow: 0 4px 15px rgba(0,0,0,0.5); z-index: 5;
            transition: opacity 0.4s ease, transform 0.4s ease; min-width: 250px;
        }
        .hidden-osd { opacity: 0 !important; pointer-events: none !important; transform: translateY(-20px); }

        #media-controls select {
            background: rgba(0, 0, 0, 0.6); border: 1px solid var(--amber); color: var(--amber);
            font-size: 0.65rem; padding: 4px; font-family: var(--font-ui); cursor: pointer; border-radius: 3px; outline: none;
        }
        #media-controls select:hover { background: var(--amber); color: #000; }

        /* --- MENU TÉCNICO OSD (NUEVO) --- */
        #osd-tech-menu {
            display: none; margin-top: 15px; padding-top: 15px;
            border-top: 1px dashed rgba(0, 243, 255, 0.3); font-family: var(--font-mono); font-size: 0.75rem;
            animation: fadeIn 0.3s ease-out;
        }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(-5px); } to { opacity: 1; transform: translateY(0); } }
        .tech-stat { color: var(--green); margin-bottom: 5px; display: flex; justify-content: space-between; }
        .tech-stat span.val { color: #fff; font-weight: bold; }
        .osd-slider-container { display: flex; align-items: center; justify-content: space-between; margin-top: 8px; }
        .osd-slider-container label { color: var(--cyan); font-weight: bold; font-size: 0.7rem; width: 70px; }
        .cyber-slider {
            -webkit-appearance: none; flex: 1; height: 4px; background: rgba(0, 243, 255, 0.2);
            outline: none; border-radius: 2px; margin-left: 10px;
        }
        .cyber-slider::-webkit-slider-thumb {
            -webkit-appearance: none; appearance: none; width: 12px; height: 12px;
            background: var(--cyan); cursor: pointer; border-radius: 50%; box-shadow: 0 0 5px var(--cyan);
        }

        /* --- EMOJIS Y REACCIONES FLOTANTES --- */
        #reaction-viewport {
            position: absolute; top: 0; left: 0; width: 100%; height: 100%;
            pointer-events: none; z-index: 90; overflow: hidden;
        }
        .floating-reaction {
            position: absolute; bottom: 85px; font-size: 2.5rem; 
            animation: floatUpEmoji 2s ease-out forwards; pointer-events: none;
            filter: drop-shadow(0 0 10px rgba(0, 243, 255, 0.5));
        }
        @keyframes floatUpEmoji {
            0% { transform: translateY(0) scale(0.5) rotate(0deg); opacity: 1; }
            50% { transform: translateY(-150px) scale(1.2) rotate(15deg); opacity: 1; }
            100% { transform: translateY(-350px) scale(1.5) rotate(-15deg); opacity: 0; }
        }

        /* --- BURBUJA INTELIGENTE DE EMOJIS --- */
        #emoji-bubble {
            position: absolute; bottom: 75px; left: 50%; transform: translateX(-50%);
            display: flex; gap: 12px; background: rgba(5, 5, 5, 0.85);
            padding: 10px 20px; border-radius: 50px; border: 1px solid var(--cyan);
            backdrop-filter: blur(10px); -webkit-backdrop-filter: blur(10px);
            box-shadow: 0 0 15px rgba(0, 243, 255, 0.3); z-index: 80;
            transition: opacity 0.4s ease, transform 0.4s ease;
        }
        .hidden-bubble { opacity: 0 !important; pointer-events: none !important; transform: translate(-50%, 20px) !important; }
        .emoji-btn {
            font-size: 1.5rem; cursor: pointer; background: none; border: none; padding: 0;
            transition: transform 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275); outline: none;
        }
        .emoji-btn:hover { transform: scale(1.4); }

        /* --- PANEL INFERIOR DE CANALES (BOTTOM BAR) --- */
        #bottom-bar {
            position: absolute; bottom: 0; left: 0; width: 100%;
            background: rgba(5, 5, 5, 0.85); border-top: 1px solid var(--cyan);
            backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px);
            display: flex; justify-content: space-between; align-items: center;
            padding: 12px 25px; z-index: 10;
            transition: opacity 0.4s ease, transform 0.4s ease;
        }
        .hidden-bottom-bar { opacity: 0 !important; pointer-events: none !important; transform: translateY(100%); }

        /* --- BOTONES TIPO CONTROL REMOTO --- */
        .remote-btn {
            background: linear-gradient(145deg, rgba(0, 243, 255, 0.1), rgba(0, 243, 255, 0.3));
            border: 2px solid var(--cyan); color: var(--cyan); font-family: var(--font-ui); 
            font-weight: 900; font-size: 1rem; padding: 10px 25px; cursor: pointer; 
            border-radius: 12px; transition: all 0.2s ease;
            box-shadow: 2px 4px 10px rgba(0,0,0,0.5), inset 0px 0px 8px rgba(0,243,255,0.2);
            flex-shrink: 0; text-shadow: 0 0 5px var(--cyan);
        }
        .remote-btn:hover {
            background: var(--cyan); color: #000; transform: translateY(-2px) scale(1.05);
            box-shadow: 0 0 15px var(--cyan), inset 0px 0px 10px rgba(255,255,255,0.5);
            text-shadow: none;
        }
        .remote-btn:active { transform: translateY(2px) scale(0.98); box-shadow: 0 0 5px var(--cyan); }

        .like-btn-tiktok {
            background: rgba(255, 0, 85, 0.1); border: 2px solid var(--red); color: var(--red);
            font-family: var(--font-ui); font-weight: bold; font-size: 0.7rem; padding: 5px 12px;
            cursor: pointer; border-radius: 4px; transition: 0.3s;
        }
        .like-btn-tiktok:hover { background: var(--red); color: #000; box-shadow: 0 0 10px var(--red); }

        /* --- RANKING CARDS --- */
        .card.rank-0 { border-left: 4px solid var(--gold); background: rgba(255, 215, 0, 0.15); }
        .card.rank-1 { border-left: 4px solid #c0c0c0; background: rgba(192, 192, 192, 0.15); }
        .card.rank-2 { border-left: 4px solid #cd7f32; background: rgba(205, 127, 50, 0.15); }

        /* --- BUSCADOR INTELIGENTE EN EL REPRODUCTOR --- */
        .in-player-search-box { position: relative; display: flex; align-items: center; }
        #in-player-search {
            background: rgba(0, 0, 0, 0.8); border: 1px solid var(--cyan); 
            color: var(--cyan); padding: 12px 15px; font-family: var(--font-mono); 
            border-radius: 8px; outline: none; transition: box-shadow 0.3s; 
            width: 100%; max-width: 400px; text-align: center; margin: 0; font-size: 0.9rem;
        }
        #in-player-search:focus { box-shadow: 0 0 10px var(--cyan-glow); }
        
        #zapper-results {
            display: none; position: absolute; bottom: calc(100% + 10px); left: 50%; 
            transform: translateX(-50%); width: 100%; max-width: 400px;
            background: rgba(5, 5, 5, 0.95); border: 1px solid var(--cyan); border-radius: 8px; 
            max-height: 200px; overflow-y: auto; z-index: 100; box-shadow: 0 0 15px rgba(0,243,255,0.3);
            backdrop-filter: blur(10px);
        }
        .zapper-item {
            padding: 10px 15px; font-size: 0.85rem; color: #fff; cursor: pointer; 
            border-bottom: 1px solid rgba(0,243,255,0.1); white-space: nowrap; 
            overflow: hidden; text-overflow: ellipsis; transition: background 0.2s;
        }
        .zapper-item:hover { background: rgba(0,243,255,0.2); }

        #admin-telemetry-panel {
            display: none; position: absolute; bottom: 100px; left: 25px; width: 350px;
            background: rgba(255, 0, 85, 0.15); border: 1px solid var(--red); border-left: 4px solid var(--red);
            padding: 15px; z-index: 15; backdrop-filter: blur(8px); border-radius: 8px; box-shadow: 0 4px 15px rgba(255, 0, 85, 0.3);
        }
        #admin-telemetry-panel .header-tel { color: #fff; font-family: var(--font-ui); font-weight: bold; font-size: 0.8rem; margin-bottom: 5px; display: flex; justify-content: space-between; }
        .pulse-red { width: 10px; height: 10px; background: var(--red); border-radius: 50%; box-shadow: 0 0 10px var(--red); animation: pulse 1s infinite alternate; }

        .sidebar { 
            position: absolute; right: 0; top: 0; height: 100%; width: 400px; 
            display: flex; flex-direction: column; 
            background: rgba(5, 5, 5, 0.6); border-left: 1px solid rgba(0, 243, 255, 0.3); 
            backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); 
            z-index: 10; transition: opacity 0.4s ease, transform 0.4s ease; 
        }
        .hidden-sidebar { opacity: 0 !important; pointer-events: none !important; transform: translateX(30px); }
        
        .ai-monitor {
            height: 140px; background: rgba(0,0,0,0.7); padding: 15px; font-size: 0.8rem;
            color: var(--green); border-bottom: 1px solid rgba(255,255,255,0.1); 
            overflow-y: hidden; display: flex; flex-direction: column-reverse;
        }

        .list-container { flex: 1; overflow-y: auto; padding: 10px; scroll-behavior: smooth; }
        
        .card {
            display: flex; align-items: center; padding: 12px 15px; margin-bottom: 8px;
            background: rgba(255,255,255,0.05); border: 1px solid rgba(255,255,255,0.08); 
            border-radius: 6px; cursor: pointer; transition: all 0.2s ease; position: relative;
        }
        .card:hover { border-color: var(--cyan); background: rgba(0, 243, 255, 0.15); transform: translateX(5px); }
        .card.active { border-left: 4px solid var(--cyan); background: rgba(0, 243, 255, 0.25); }
        .card.dead { opacity: 0.3; filter: grayscale(1); order: 100; }

        .status-dot { width: 10px; height: 10px; border-radius: 50%; margin-right: 15px; flex-shrink: 0; }
        .st-live { background: var(--green); box-shadow: 0 0 10px var(--green); }
        .st-dead { background: var(--red); box-shadow: 0 0 10px var(--red); }
        .st-unk { background: var(--amber); }

        .controls { padding: 15px; background: rgba(0,0,0,0.5); border-bottom: 1px solid rgba(255,255,255,0.05); }

        /* --- PAGINACION ESTILOS --- */
        .pagination-controls {
            display: flex; justify-content: space-between; align-items: center;
            padding: 10px 15px; background: rgba(0,0,0,0.6); 
            border-top: 1px solid rgba(0, 243, 255, 0.2); border-bottom: 1px solid rgba(0, 243, 255, 0.2);
        }
        .page-btn {
            background: rgba(0, 243, 255, 0.1); border: 1px solid var(--cyan); color: var(--cyan);
            padding: 5px 12px; cursor: pointer; border-radius: 4px; font-family: var(--font-ui); font-size: 0.75rem; font-weight: bold;
            transition: all 0.2s;
        }
        .page-btn:hover:not(:disabled) { background: var(--cyan); color: #000; box-shadow: 0 0 8px var(--cyan); }
        .page-btn:disabled { opacity: 0.3; cursor: not-allowed; border-color: #555; color: #555; }
        
        /* --- MODALES GENERALES --- */
        .modal-overlay {
            display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(0, 0, 0, 0.85); backdrop-filter: blur(5px); z-index: 100;
            justify-content: center; align-items: center;
        }
        .modal-box {
            background: var(--bg-dark); border: 1px solid var(--amber);
            box-shadow: 0 0 20px rgba(255, 174, 0, 0.3); padding: 30px; width: 350px; max-width: 95%;
            border-radius: 8px; text-align: center; max-height: 90vh; overflow-y: auto;
        }
        .modal-box h2 { font-family: var(--font-ui); color: var(--amber); margin-top: 0; font-size: 1.2rem; letter-spacing: 2px; }
        .btn-submit {
            width: 100%; background: var(--amber); color: #000; border: none; padding: 10px;
            font-family: var(--font-ui); font-weight: bold; cursor: pointer;
            border-radius: 4px; margin-top: 10px; transition: transform 0.2s;
        }
        .btn-submit.register { background: var(--cyan); }
        .btn-submit:hover { transform: scale(1.02); box-shadow: 0 0 10px currentColor; }
        .close-modal { color: #888; font-size: 0.8rem; cursor: pointer; margin-top: 15px; display: inline-block; }
        .close-modal:hover { color: #fff; }

        /* PANEL ADMIN OCULTO */
        #admin-tools {
            display: none; background: rgba(255, 174, 0, 0.1); border-bottom: 1px solid var(--amber);
            padding: 10px 15px; font-size: 0.85rem; max-height: 50vh; overflow-y: auto;
        }
        .admin-list { margin-top: 10px; border-top: 1px dashed var(--amber); padding-top: 10px; max-height: 100px; overflow-y: auto; }
        .user-req { display: flex; justify-content: space-between; align-items: center; margin-bottom: 5px; color: #fff; }

        /* --- TARJETA DE PRESENTACIÓN --- */
        .id-card {
            background: rgba(0, 243, 255, 0.05); border: 1px solid var(--cyan); border-radius: 8px;
            padding: 15px; margin-top: 20px; display: flex; align-items: center; text-align: left;
            box-shadow: inset 0 0 10px rgba(0, 243, 255, 0.2); position: relative; overflow: hidden;
        }
        .id-card::before {
            content: "CORTEX_ID"; position: absolute; top: -5px; right: -10px; font-size: 3rem;
            color: rgba(0, 243, 255, 0.05); font-family: var(--font-ui); font-weight: 900;
            transform: rotate(-10deg); z-index: 0;
        }
        .id-photo { width: 80px; height: 80px; border-radius: 4px; border: 2px solid var(--cyan); object-fit: cover; margin-right: 15px; background: #000; z-index: 1; }
        .id-info { flex: 1; z-index: 1; }
        .id-info .label { font-size: 0.6rem; color: var(--cyan); margin-bottom: 2px; }
        .id-info .data { font-size: 0.9rem; font-family: var(--font-ui); margin-bottom: 8px; font-weight: bold;}
        .id-role { display: inline-block; background: var(--amber); color: #000; font-size: 0.6rem; padding: 2px 6px; border-radius: 2px; font-weight: bold; }
        .input-file-label { display: block; width: 100%; background: rgba(255,255,255,0.1); border: 1px dashed var(--cyan); padding: 10px; color: var(--cyan); cursor: pointer; margin-bottom: 10px; font-size: 0.8rem; }

        /* --- CHAT OVERLAY --- */
        #chat-overlay {
            position: absolute; top: 0; right: 0; height: 100%; width: 320px;
            background: rgba(5, 5, 5, 0.85); border-left: 1px solid var(--cyan); display: none; 
            flex-direction: column; z-index: 60; backdrop-filter: blur(5px);
            box-shadow: -5px 0 15px rgba(0,0,0,0.5); transform: translateX(0);
        }
        .chat-header {
            padding: 15px; background: rgba(0, 243, 255, 0.1); border-bottom: 1px solid var(--cyan);
            color: var(--cyan); font-family: var(--font-ui); font-size: 0.9rem; display: flex; justify-content: space-between; align-items: center;
        }
        #chat-messages { flex: 1; overflow-y: auto; padding: 15px; display: flex; flex-direction: column; gap: 10px; }
        .chat-input-box { display: flex; padding: 10px; border-top: 1px solid rgba(0, 243, 255, 0.2); background: #000; }
        .chat-input-box input { flex: 1; margin: 0; border: none; border-bottom: 1px solid var(--cyan); background: transparent; border-radius: 0; padding: 8px; outline: none; color: var(--cyan); }
        .chat-input-box button { background: var(--cyan); color: #000; border: none; font-weight: bold; cursor: pointer; padding: 0 15px; font-family: var(--font-mono); margin-left: 5px; }
        .msg-line { font-size: 0.85rem; word-wrap: break-word; line-height: 1.3; }
        .msg-user { color: var(--amber); font-weight: bold; }
        .msg-user.admin { color: var(--red); }
        .msg-time { font-size: 0.6rem; color: #666; margin-left: 5px; }

        /* --- BURBUJA AGENTE IA --- */
        .ai-bubble {
            position: absolute; top: 50%; left: 50%; transform: translate(-50%, -50%);
            display: flex; flex-direction: column; align-items: center; justify-content: center;
            background: rgba(10, 15, 20, 0.9); border: 2px solid var(--amber); border-radius: 15px;
            padding: 20px 40px; box-shadow: 0 0 30px rgba(255, 174, 0, 0.5); z-index: 20; text-align: center;
            backdrop-filter: blur(10px); animation: pulse-bubble 2s infinite alternate;
        }
        @keyframes pulse-bubble { 0% { box-shadow: 0 0 20px rgba(255, 174, 0, 0.3); } 100% { box-shadow: 0 0 50px rgba(255, 174, 0, 0.8); } }
        .ai-bubble .loader { width: 50px; height: 50px; border: 4px solid transparent; border-top-color: var(--amber); border-bottom-color: var(--amber); border-radius: 50%; animation: spin 1s linear infinite; margin-bottom: 15px; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
        .ai-bubble-text { font-family: var(--font-ui); color: var(--amber); font-weight: bold; font-size: 1.1rem; }
        .ai-bubble-sub { font-size: 0.8rem; color: #fff; margin-top: 5px; }

        /* --- WIKIPEDIA ESTILOS --- */
        .wiki-section { margin-bottom: 15px; }
        .wiki-section h3 { color: var(--green); border-bottom: 1px solid rgba(10,255,0,0.3); padding-bottom: 5px; margin-bottom: 8px; font-size: 1rem; }
        .wiki-section p, .wiki-section ul, .wiki-section table { font-family: sans-serif; font-size: 0.85rem; color: #ddd; margin-bottom: 10px; line-height: 1.6; }
        .wiki-section code { background: rgba(255,255,255,0.1); padding: 2px 5px; border-radius: 3px; font-family: var(--font-mono); color: var(--cyan);}
        .code-block { background: #000; border: 1px solid var(--cyan); border-left: 4px solid var(--cyan); padding: 10px; font-family: var(--font-mono); font-size: 0.8rem; color: var(--green); overflow-x: auto; margin-bottom: 10px; }

        /* SCROLLBARS */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: rgba(0,0,0,0.3); }
        ::-webkit-scrollbar-thumb { background: var(--cyan); border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: #fff; }

        /* --- PANEL FLOTANTE MÓVIL --- */
        #mobile-floating-node { display: none; }
        .pulse-dot { display: inline-block; width: 8px; height: 8px; background: var(--green); border-radius: 50%; box-shadow: 0 0 8px var(--green); animation: pulse 1.5s infinite; margin-right: 5px; }

        /* --- RESPONSIVE --- */
        @media (max-width: 900px) {
            html, body { overflow: auto !important; height: auto !important; }
            header { flex-direction: column; gap: 10px; padding: 10px 5px; height: auto; }
            .ai-status { flex-wrap: wrap; justify-content: center; gap: 5px; }
            .main { flex-direction: column; height: auto !important; overflow: visible !important; }
            
            .player-box { 
                flex: none; height: 35vh; width: 100%; min-height: 250px; 
                position: -webkit-sticky !important; position: sticky !important; 
                top: 0; z-index: 150 !important; box-shadow: 0 10px 25px rgba(0,0,0,0.9); border-bottom: 1px solid var(--cyan);
            }

            .sidebar { position: relative; width: 100%; flex: auto; height: auto !important; border-left: none; border-top: none; background: var(--bg-dark); }
            .list-container { overflow-y: visible !important; height: auto !important; flex: auto; }
            .hidden-sidebar { transform: none !important; opacity: 1 !important; pointer-events: auto !important; } 
            .hidden-osd { transform: translateY(-20px) !important; opacity: 0 !important; pointer-events: none !important; }
            .hidden-bottom-bar { transform: translateY(100%) !important; opacity: 0 !important; pointer-events: none !important; }
            .hidden-bubble { opacity: 0 !important; pointer-events: none !important; transform: translate(-50%, 20px) !important; }
            
            #floating-max-btn { top: 15px; right: 15px; font-size: 0.85rem; padding: 6px 12px; }
            #emoji-bubble { bottom: 65px; padding: 5px 15px; gap: 10px; }
            .emoji-btn { font-size: 1.3rem; }
            .osd { top: 5px; left: 5px; padding: 10px; }
            #osd-name { font-size: 1rem !important; }
            #user-action-btns { flex-wrap: wrap; gap: 4px; }
            #user-action-btns button { font-size: 0.55rem !important; padding: 4px 6px; }
            #bottom-bar { flex-direction: column; gap: 10px; padding: 10px; text-align: center; }
            #bottom-bar > div:last-child { justify-content: space-between; width: 100%; flex-wrap: wrap; padding: 0;}
            .remote-btn { font-size: 0.8rem; padding: 10px; }
            .in-player-search-box { width: 100%; margin: 10px 0; }
            #chat-overlay { width: 100%; } 
            .ai-bubble { padding: 10px 15px; }
            .ai-bubble-text { font-size: 0.9rem; }
            .ai-bubble .loader { width: 30px; height: 30px; }
            #admin-telemetry-panel { width: 90%; right: 5%; left: 5%; bottom: 120px; padding: 10px; }
            .modal-box { width: 95%; padding: 20px; }
            select, .btn-admin { font-size: 0.75rem; padding: 6px 10px; }

            #mobile-floating-node.active {
                display: flex; flex-direction: column; position: fixed; bottom: 20px; right: 20px;
                background: rgba(5, 5, 5, 0.95); border: 1px solid var(--cyan); border-left: 4px solid var(--cyan);
                box-shadow: 0 4px 15px rgba(0, 0, 0, 0.8), 0 0 15px rgba(0, 243, 255, 0.3); 
                padding: 10px 15px; border-radius: 8px; z-index: 9999; cursor: pointer; max-width: 65%; 
                backdrop-filter: blur(8px); -webkit-backdrop-filter: blur(8px); transition: transform 0.2s ease;
            }
            #mobile-floating-node.active:active { transform: scale(0.95); }
            #mobile-floating-node .node-badge { font-size: 0.65rem; color: var(--green); font-family: var(--font-ui); font-weight: bold; margin-bottom: 4px; display: flex; align-items: center; }
            #mobile-floating-name { font-size: 0.85rem; color: var(--cyan); white-space: nowrap; overflow: hidden; text-overflow: ellipsis; font-weight: bold; font-family: var(--font-mono); }
        }
    </style>
</head>
<body>

<header>
    <div class="ai-brain">
        <div class="pulse"></div>
        <div style="font-family: var(--font-ui); font-weight: 900; letter-spacing: 2px; font-size: 1.2rem;">
            CORTEX_V15 // <span style="color: var(--cyan);">HIVE</span>
        </div>
        <div id="user-timer" style="display:none; align-items:center; gap:8px; background:rgba(0,255,0,0.1); border:1px solid var(--green); padding:4px 12px; border-radius:4px; margin-left: 20px;">
            <i class="fa-solid fa-stopwatch" style="color:var(--green);"></i>
            <span style="color:var(--green); font-family:var(--font-mono); font-size:0.85rem; font-weight:bold;"><span id="timer-days">0</span> DÍAS RESTANTES</span>
        </div>
    </div>
    <div class="ai-status">
        <span id="ai-count" style="color:var(--cyan); font-size: 0.9rem;">ESPERANDO DATOS...</span>
        <select id="region" onchange="changeRegion(this.value)">
            <option value="index">🌐 RED GLOBAL (TODAS LAS CÁMARAS)</option>
            <option value="index_nsfw">🔞 RED GLOBAL (+18)</option>
            <option value="co">🇨🇴 COLOMBIA</option>
            <option value="mx">🇲🇽 MEXICO</option>
            <option value="es">🇪🇸 ESPAÑA</option>
            <option value="ar">🇦🇷 ARGENTINA</option>
            <option value="cat_sports">⚽ DEPORTES</option>
            <option value="cat_movies">🎬 CINE</option>
            <option value="cat_news">📰 NOTICIAS</option>
        </select>
        <button class="btn-admin register" id="btn-profile" onclick="openProfile()" style="display:none;">MI PERFIL</button>
        <button class="btn-admin" id="btn-open-login" onclick="openLogin()">SISTEMA</button>
    </div>
</header>

<div class="modal-overlay" id="login-modal">
    <div class="modal-box">
        <h2>ACCESO AL SISTEMA</h2>
        <input type="email" id="auth-email" placeholder="IDENTIFICACIÓN (Correo)" style="width:100%; margin-bottom:10px; padding:10px; background:rgba(0,0,0,0.5); border:1px solid var(--cyan); color:#fff;">
        <input type="password" id="auth-pass" placeholder="CÓDIGO DE SEGURIDAD" style="width:100%; margin-bottom:10px; padding:10px; background:rgba(0,0,0,0.5); border:1px solid var(--cyan); color:#fff;">
        <input type="text" id="auth-keyword" placeholder="PALABRA CLAVE (Solo para Registro/Recuperar)" style="width:100%; margin-bottom:10px; padding:10px; background:rgba(0,0,0,0.5); border:1px solid var(--amber); color:#fff;">
        
        <button class="btn-submit" onclick="loginUser()">INICIAR SESIÓN</button>
        <button class="btn-submit register" onclick="registerUser()">SOLICITAR REGISTRO</button>
        <button class="btn-submit" style="background:rgba(255, 0, 85, 0.2); border: 1px solid var(--red); color: var(--red);" onclick="recoverPassword()">RECUPERAR CÓDIGO</button>
        <div class="close-modal" onclick="closeLogin()">[ ABORTAR CONEXIÓN ]</div>
    </div>
</div>

<div class="modal-overlay" id="profile-modal">
    <div class="modal-box" style="width: 400px; border-color: var(--cyan);">
        <h2 style="color: var(--cyan);">BASE DE DATOS DE PERSONAL</h2>
        <input type="text" id="prof-name" placeholder="NOMBRE DE OPERADOR" style="width:100%; margin-bottom:10px; padding:10px; background:rgba(0,0,0,0.5); border:1px solid var(--cyan); color:#fff;">
        <label class="input-file-label">
            SELECCIONAR FOTO DE IDENTIFICACIÓN
            <input type="file" id="prof-photo-input" accept="image/*" style="display:none;" onchange="handlePhotoUpload(event)">
        </label>
        <button class="btn-submit register" onclick="saveProfile()">ACTUALIZAR DATOS</button>
        <div class="id-card">
            <img src="https://via.placeholder.com/80/000000/00f3ff?text=SYS" alt="Foto" class="id-photo" id="card-photo">
            <div class="id-info">
                <div class="label">NIVEL DE ACCESO</div>
                <div class="id-role" id="card-role">USUARIO</div>
                <div class="label" style="margin-top:5px;">IDENTIFICACIÓN</div>
                <div class="data" id="card-name">DESCONOCIDO</div>
                <div class="label">ENLACE COMUNICACIÓN</div>
                <div class="data" id="card-email" style="font-size:0.7rem;">correo@sistema.net</div>
            </div>
        </div>
        <div class="close-modal" onclick="closeProfile()">[ CERRAR PANEL ]</div>
    </div>
</div>

<div class="modal-overlay" id="av1-info-modal">
    <div class="modal-box" style="width: 85%; max-width: 800px; border-color: var(--cyan); text-align: left;">
        <h2 style="color: var(--cyan); text-align: center; margin-bottom: 20px;">[ EXPEDIENTE CLASIFICADO: AOMedia Video 1 ]</h2>
        <div class="wiki-section">
            <h3>CÓDEC AV1 (Descripción Base)</h3>
            <p><strong>AOMedia Video 1 (AV1)</strong> es un formato de codificación de vídeo abierto y libre de regalías, diseñado inicialmente para transmisiones de vídeo por Internet.</p>
        </div>
        <div class="wiki-section">
            <h3>TECNOLOGÍA (Particionado y Predicción)</h3>
            <p>AV1 es un formato tradicional de transformación de frecuencia basado en bloques que incorpora nuevas técnicas. Basado en el VP9 de Google, AV1 incorpora técnicas adicionales que brindan a los codificadores más opciones de codificación.</p>
        </div>
        <div style="text-align: center; margin-top: 25px;">
            <button class="btn-submit" onclick="document.getElementById('av1-info-modal').style.display='none'" style="width: auto; padding: 10px 30px;">[ CERRAR EXPEDIENTE ]</button>
        </div>
    </div>
</div>

<div class="modal-overlay" id="iptv-docs-modal">
    <div class="modal-box" style="width: 90%; max-width: 900px; border-color: var(--amber); text-align: left;">
        <h2 style="color: var(--amber); text-align: center; margin-bottom: 20px;">[ DOCUMENTACIÓN TÉCNICA: IPTV-ORG ]</h2>
        <div class="wiki-section">
            <h3>¿Cómo encontrar una transmisión rota?</h3>
            <div class="code-block">npm run playlist:test path/to/playlist.m3u</div>
            <p>Este comando ejecutará una verificación automática de todos los enlaces en la lista de reproducción y mostrará su estado. Además, si agrega la opción <code>--fix</code> al comando, el script eliminará automáticamente todos los flujos rotos que encuentre.</p>
        </div>
        <div class="wiki-section">
            <h3>Esquema de descripción de transmisión (M3U)</h3>
            <div class="code-block">#EXTINF:-1 tvg-id="STREAM_ID",STREAM_TITLE (QUALITY) [LABEL]<br>STREAM_URL</div>
        </div>
        <div style="text-align: center; margin-top: 25px;">
            <button class="btn-submit" onclick="document.getElementById('iptv-docs-modal').style.display='none'" style="width: auto; padding: 10px 30px;">[ CERRAR DOCUMENTACIÓN ]</button>
        </div>
    </div>
</div>

<div class="modal-overlay" id="tivimate-docs-modal">
    <div class="modal-box" style="width: 90%; max-width: 900px; border-color: var(--purple); text-align: left;">
        <h2 style="color: var(--purple); text-align: center; margin-bottom: 20px;">[ GUÍA DE REPRODUCCIÓN EXTERNA: TIVIMATE ]</h2>
        <div class="wiki-section">
            <h3>REPRODUCTOR RECOMENDADO</h3>
            <p>Lo primero de todo pasa por tener instalado un reproductor compatible. Existen muchas aplicaciones de IPTV en Android TV, por lo que a priori valdría con cualquiera. Para este manual ilustraremos con <strong>'TiviMate IPTV Player'</strong>, una app disponible en Google Play que, aunque tiene opciones de pago, es gratuita para esto y ofrece una interfaz muy sencilla e intuitiva.</p>
        </div>
        <div class="wiki-section">
            <h3>ENLACES DE MATRIZ (M3U)</h3>
            <p>Una vez tengas la aplicación en tu televisor, necesitarás a mano la lista de canales. Dependiendo de si quieres contenido explícito o no, la matriz utiliza:</p>
            <ul>
                <li><strong>Canales para todos los públicos:</strong> <code>https://iptv-org.github.io/iptv/index.m3u</code></li>
                <li><strong>Todos + Adultos (+18):</strong> <code>https://iptv-org.github.io/iptv/index.nsfw.m3u</code></li>
            </ul>
        </div>
        <div class="wiki-section">
            <h3>PROTOCOLO DE CONFIGURACIÓN</h3>
            <p>
                1. Abre 'TiviMate' en tu televisor con Android TV.<br>
                2. Nada más abrirla, te dará la opción de <strong>'Añadir lista'</strong>.<br>
                3. Elige la opción <strong>'Lista M3U'</strong>.<br>
                4. Añade la URL de la lista en el campo de texto. <i>(Consejo: recomendamos usar la app de 'Google Home' en el móvil para escribir con el teclado e incluso pegar la lista fácilmente).</i><br>
                5. Pasados unos segundos del proceso de añadido, pulsa en <strong>'Siguiente'</strong>.<br>
                6. Finalmente, pulsa en <strong>'Listo'</strong>.<br><br>
                Una vez completado el protocolo, tendrás siempre disponible este listado con una organización impecable. Podrás desplegar el menú de la izquierda para acceder fácilmente a todas las categorías.
            </p>
        </div>
        <div style="text-align: center; margin-top: 25px;">
            <button class="btn-submit" onclick="document.getElementById('tivimate-docs-modal').style.display='none'" style="background: rgba(208, 0, 255, 0.2); border: 1px solid var(--purple); color: var(--purple); width: auto; padding: 10px 30px;">[ CERRAR GUÍA EXTERNA ]</button>
        </div>
    </div>
</div>

<div class="main">
    <div class="player-box" id="main-player-box">
        
        <button id="floating-max-btn" onclick="toggleFullScreen()" title="PANTALLA COMPLETA">
            <i class="fa-solid fa-expand"></i> MAX
        </button>

        <div id="reaction-viewport"></div>

        <div class="osd" id="ui-osd">
            <div id="osd-name" style="color:var(--cyan); font-family: var(--font-ui); font-weight: bold; font-size: 1.4rem; letter-spacing: 1px;">SISTEMA INICIADO</div>
            <div id="osd-msg" style="font-size: 0.9rem; opacity: 0.8; margin-top: 5px;">ESPERANDO IDENTIFICACIÓN...</div>
            <div id="osd-res" style="font-size: 0.7rem; color: var(--amber); margin-top: 3px; font-weight: bold; display: none;">RESOLUCIÓN: CALCULANDO...</div>
            
            <div id="user-action-btns" style="display:none; margin-top:10px; gap:5px; flex-wrap:wrap;">
                <button class="like-btn-tiktok" onclick="sendReaction('❤️')">[ ❤️ LIKE ]</button>
                <button onclick="toggleChat()" style="background:rgba(0, 243, 255, 0.1); border:1px solid var(--cyan); color:var(--cyan); font-family:var(--font-ui); font-size:0.7rem; padding:5px 10px; cursor:pointer; font-weight:bold;">[ COMUNICADOR ]</button>
                <button onclick="addCurrentToFavs()" style="background:rgba(255, 174, 0, 0.1); border:1px solid var(--amber); color:var(--amber); font-family:var(--font-ui); font-size:0.7rem; padding:5px 10px; cursor:pointer; font-weight:bold;">[ ⭐ FAV ]</button>
                <button onclick="addCurrentToSaved()" style="background:rgba(10, 255, 0, 0.1); border:1px solid var(--green); color:var(--green); font-family:var(--font-ui); font-size:0.7rem; padding:5px 10px; cursor:pointer; font-weight:bold;">[ 💾 GUARDAR SEÑAL ]</button>
                <button onclick="toggleOSDMenu()" style="background:rgba(208, 0, 255, 0.1); border:1px solid var(--purple); color:var(--purple); font-family:var(--font-ui); font-size:0.7rem; padding:5px 10px; cursor:pointer; font-weight:bold;">[ ⚙️ CFG OSD ]</button>
                <button id="btn-av1-info" onclick="document.getElementById('av1-info-modal').style.display='flex'" style="display:none; background:rgba(255, 255, 255, 0.1); border:1px solid #fff; color:#fff; font-family:var(--font-ui); font-size:0.7rem; padding:5px 10px; cursor:pointer; font-weight:bold;">[ 📄 DOCS AV1 ]</button>
            </div>

            <div id="osd-tech-menu">
                <div class="tech-stat"><span>WDR (Rango Dinámico):</span><span class="val" id="stat-wdr">AUTO</span></div>
                <div class="tech-stat"><span>DNR (Reducción Ruido):</span><span class="val">ON</span></div>
                <div class="tech-stat"><span>SALUD DE BUFFER:</span><span class="val" id="stat-buffer">100%</span></div>
                
                <div style="margin-top: 10px;">
                    <div class="osd-slider-container">
                        <label>VOLUMEN</label>
                        <input type="range" id="osd-vol" class="cyber-slider" min="0" max="1" step="0.05" value="1" oninput="updateOSDSettings()">
                    </div>
                    <div class="osd-slider-container">
                        <label>BRILLO</label>
                        <input type="range" id="osd-bri" class="cyber-slider" min="0.5" max="1.5" step="0.05" value="1" oninput="updateOSDSettings()">
                    </div>
                    <div class="osd-slider-container">
                        <label>CONTRASTE</label>
                        <input type="range" id="osd-con" class="cyber-slider" min="0.5" max="1.5" step="0.05" value="1" oninput="updateOSDSettings()">
                    </div>
                </div>
            </div>

            <div id="media-controls" style="display:none; margin-top: 8px; gap: 5px;">
                <select id="audio-tracks" onchange="changeAudioTrack(this.value)" style="display:none; background:rgba(0,0,0,0.7); border:1px solid var(--cyan); color:var(--cyan); font-size:0.65rem; padding:3px; font-weight:bold;"></select>
                <select id="subtitle-tracks" onchange="changeSubtitleTrack(this.value)" style="display:none; background:rgba(0,0,0,0.7); border:1px solid var(--amber); color:var(--amber); font-size:0.65rem; padding:3px; font-weight:bold;"></select>
            </div>
        </div>

        <div id="emoji-bubble" class="hidden-bubble">
            <button class="emoji-btn" onclick="sendReaction('🔥')">🔥</button>
            <button class="emoji-btn" onclick="sendReaction('👏')">👏</button>
            <button class="emoji-btn" onclick="sendReaction('😂')">😂</button>
            <button class="emoji-btn" onclick="sendReaction('😍')">😍</button>
            <button class="emoji-btn" onclick="sendReaction('⚡')">⚡</button>
            <button class="emoji-btn" onclick="sendReaction('🦾')">🦾</button>
            <button class="emoji-btn" onclick="sendReaction('🚀')">🚀</button>
        </div>
        
        <div id="ai-bubble" class="ai-bubble" style="display:none;">
            <div class="loader"></div>
            <div class="ai-bubble-text">AGENTE IA EN LÍNEA</div>
            <div class="ai-bubble-sub" id="ai-bubble-msg">ANALIZANDO PÉRDIDA DE SEÑAL...</div>
        </div>

        <video id="video-core" controls autoplay preload="metadata" playsinline>Tu navegador no soporta el elemento de video.</video>

        <video id="video-av1-panel" controls width="100%" poster="imagen-previa.jpg" playsinline style="display:none; object-fit: contain;">
          <source src="video-av1.mp4" type='video/mp4; codecs="av01.0.05M.08"'>
          <source src="video-vp9.webm" type="video/webm; codecs=vp9">
          <source src="video-h264.mp4" type="video/mp4">
          Tu navegador no soporta la reproducción de video.
        </video>

        <div id="bottom-bar">
            <div style="flex: 1; display: flex; flex-direction: column;">
                <div style="display: inline-block; width: 100%;">
                    <div id="bb-channel-name" style="color: var(--cyan); font-family: var(--font-ui); font-weight: bold; font-size: 1.1rem; overflow:hidden; text-overflow:ellipsis; white-space:nowrap; display: inline-block; vertical-align: middle; max-width: 100%;">SISTEMA EN ESPERA</div>
                </div>
                <div id="bb-program-info" style="color: #aaa; font-size: 0.8rem; margin-top: 3px; font-family: var(--font-mono);">INFO: Selecciona una cámara del directorio...</div>
            </div>
            
            <div style="display: flex; width: 100%; justify-content: space-between; align-items: center; margin-top: 5px; position: relative;">
                
                <div id="ia-sync-status" style="display:none; position:absolute; top:-25px; left:50%; transform:translateX(-50%); color: var(--green); font-size: 0.75rem; font-weight: bold; animation: pulse 1s infinite alternate; font-family: var(--font-ui);">
                    <i class="fa-solid fa-satellite-dish"></i> IA SYNC: ACTIVO
                </div>
                
                <button class="remote-btn" onclick="playPrevChannel()">[ &lt; CH - ]</button>
                
                <div class="in-player-search-box" style="flex: 1; display: flex; justify-content: center; padding: 0 15px;">
                    <input type="text" id="in-player-search" placeholder="🔎 BUSCAR CÁMARA O #..." oninput="handleInPlayerSearch()" onkeypress="handleInPlayerSearchKey(event)">
                    <div id="zapper-results"></div>
                </div>

                <button class="remote-btn" onclick="playNextChannel()">[ CH + &gt; ]</button>
            </div>
        </div>

        <div id="admin-telemetry-panel">
            <div class="header-tel">
                <span style="color: var(--red);">[ DIAGNÓSTICO DE TRANSMISIÓN ]</span>
                <span class="pulse-red"></span>
            </div>
            <div id="telemetry-channel" style="color: var(--amber); font-weight: bold; font-size: 0.75rem; margin-bottom: 5px;">NODO: ---</div>
            <div id="telemetry-status" style="color: #fff; font-size: 0.7rem; margin-bottom: 5px;">ESTADO: <span style="color:var(--amber);">EVALUANDO RENDIMIENTO...</span></div>
            <div id="telemetry-error" style="color: var(--red); font-size: 0.75rem; word-wrap: break-word; font-weight: bold; margin-top: 10px;"></div>
        </div>

        <div id="chat-overlay">
            <div class="chat-header">
                <span id="chat-title">[ ENLACE COM ]</span>
                <span style="cursor:pointer; color: var(--red);" onclick="toggleChat()">[ X ]</span>
            </div>
            <div id="chat-messages"></div>
            <div class="chat-input-box">
                <input type="text" id="chat-input" placeholder="> TRANSMITIR MENSAJE..." onkeypress="handleChatPress(event)">
                <button onclick="sendChatMessage()">></button>
            </div>
        </div>
    </div>

    <div class="sidebar">
        <div id="user-personal-panels" style="display:none; flex-direction:column; gap:8px; padding:10px 15px; border-bottom:1px solid rgba(0,243,255,0.2);">
            <button class="btn-submit" style="margin-top:0; background:rgba(255, 215, 0, 0.15); border-color:var(--gold); color:var(--gold); font-size:0.75rem;" onclick="openSubPanel('ranking')">🏆 TOP CANALES (RANKING SEMANAL)</button>
            <button class="btn-submit" style="margin-top:0; background:rgba(208, 0, 255, 0.15); border-color:var(--purple); color:var(--purple); font-size:0.75rem;" onclick="runUserAIspection()">👁️ INSPECCIÓN IA (AUTO-GUARDAR ACTIVOS)</button>
            <button class="btn-submit" style="margin-top:0; background:rgba(255, 174, 0, 0.15); border-color:var(--amber); color:var(--amber); font-size:0.75rem;" onclick="openSubPanel('favs')">⭐ MIS NODOS PREFERIDOS</button>
            <button class="btn-submit" style="margin-top:0; background:rgba(10, 255, 0, 0.15); border-color:var(--green); color:var(--green); font-size:0.75rem;" onclick="openSubPanel('saved')">💾 MIS SEÑALES GUARDADAS</button>
            <button class="btn-submit" style="margin-top:0; background:rgba(0, 243, 255, 0.15); border-color:var(--cyan); color:var(--cyan); font-size:0.75rem;" onclick="openSubPanel('auto')">📡 SEÑALES ACTIVAS (AUTO)</button>
            <button class="btn-submit" style="margin-top:0; background:rgba(255, 255, 255, 0.1); border-color:#fff; color:#fff; font-size:0.75rem;" onclick="openWorldDirectory()">🌍 DIRECTORIO MUNDIAL</button>
        </div>

        <div id="admin-tools">
            <div style="color: var(--amber); font-family: var(--font-ui); font-weight: bold; margin-bottom: 5px;">[ PANEL DE CONTROL DE SEÑAL ]</div>
            <button class="btn-submit" style="padding: 5px; font-size: 0.8rem; margin-top:0;" onclick="runAiSupervisor()">EJECUTAR IA SUPERVISOR (REPARAR/GUARDAR)</button>
            <button class="btn-submit" style="padding: 5px; font-size: 0.8rem; margin-top:5px; background: rgba(255,0,85,0.2); border: 1px solid var(--red); color: var(--red);" onclick="viewErrorDB()">LEER REGISTRO DE ERRORES (BD)</button>

            <div style="color: var(--cyan); font-family: var(--font-ui); font-weight: bold; margin-top: 15px; margin-bottom: 5px;">[ MÓDULOS IA AVANZADOS ]</div>
            <button class="btn-submit" style="padding: 5px; font-size: 0.8rem; margin-top:0; background: rgba(0, 243, 255, 0.2); border-color: var(--cyan); color: var(--cyan);" onclick="runAiSearch()">BUSCADOR IA (NUEVAS SEÑALES)</button>
            <button class="btn-submit" id="btn-monitor-ia" style="padding: 5px; font-size: 0.8rem; margin-top:5px; background: rgba(10, 255, 0, 0.2); border-color: var(--green); color: var(--green);" onclick="toggleAiMonitor()">ACTIVAR SUPERVISOR IA EN TIEMPO REAL</button>
            <button class="btn-submit" style="padding: 5px; font-size: 0.8rem; margin-top:5px; background: rgba(255, 255, 255, 0.2); border-color: #fff; color: #fff;" onclick="document.getElementById('iptv-docs-modal').style.display='flex'">LEER MANUAL DE REPOSITORIO IPTV</button>
            <button class="btn-submit" style="padding: 5px; font-size: 0.8rem; margin-top:5px; background: rgba(208, 0, 255, 0.2); border-color: var(--purple); color: var(--purple);" onclick="document.getElementById('tivimate-docs-modal').style.display='flex'">GUÍA EXTERNA: TIVIMATE (ANDROID TV)</button>

            <div style="display:flex; gap:10px; margin-top:15px;">
                <div style="flex:1; background:rgba(10,255,0,0.1); border:1px solid var(--green); padding:5px; border-radius:4px;">
                    <div style="color:var(--green); font-size:0.7rem; text-align:center; font-weight:bold;">NODOS ESTABLES (BD)</div>
                    <div id="panel-live" style="max-height:80px; overflow-y:auto; font-size:0.65rem; margin-top:5px; color:#fff;"></div>
                </div>
                <div style="flex:1; background:rgba(255,0,85,0.1); border:1px solid var(--red); padding:5px; border-radius:4px;">
                    <div style="color:var(--red); font-size:0.7rem; text-align:center; font-weight:bold;">NODOS CAÍDOS (BD)</div>
                    <div id="panel-dead" style="max-height:80px; overflow-y:auto; font-size:0.65rem; margin-top:5px; color:#fff;"></div>
                </div>
            </div>

            <div style="margin-top: 15px; border-top: 1px dashed var(--green); padding-top: 10px;">
                <div style="color: var(--green); font-family: var(--font-ui); font-size: 0.75rem; font-weight: bold; margin-bottom: 5px;">USUARIOS PERMITIDOS (BD ESTABLE)</div>
                <div id="approved-users-list" style="max-height: 80px; overflow-y: auto; font-size: 0.7rem; color: #ccc;"></div>
            </div>

            <div class="admin-list" id="pending-users"></div>

            <div style="margin-top: 15px; border-top: 1px dashed var(--red); padding-top: 10px;">
                <div style="color: var(--red); font-family: var(--font-ui); font-size: 0.75rem; font-weight: bold; margin-bottom: 5px;">USUARIOS BLOQUEADOS</div>
                <div id="blocked-users-list" style="max-height: 80px; overflow-y: auto; font-size: 0.7rem; color: #ccc;"></div>
            </div>
        </div>

        <div id="ai-user-panel" style="display: none; background: rgba(255, 174, 0, 0.1); border-bottom: 1px solid var(--amber); padding: 12px 15px;">
            <div style="display: flex; align-items: center; gap: 10px; color: var(--amber); font-family: var(--font-ui); font-weight: bold; font-size: 0.85rem; margin-bottom: 5px;">
                <div class="pulse" style="background: var(--amber); box-shadow: 0 0 10px var(--amber);"></div>
                [ AGENTE IA EN LÍNEA ]
            </div>
            <div id="ai-user-panel-msg" style="font-size: 0.75rem; color: #fff; font-family: var(--font-mono);">ANALIZANDO ANOMALÍA EN EL NODO...</div>
        </div>

        <div class="controls" id="main-controls">
            <input type="text" id="search" placeholder="> FILTRAR CÁMARA POR NOMBRE..." oninput="render(true)" style="margin-bottom:0; width:100%; padding:10px; background:rgba(0,0,0,0.8); border:1px solid var(--cyan); color:var(--cyan); border-radius:4px; outline:none;">
        </div>

        <div class="list-container" id="list">
            </div>

        <div class="pagination-controls" id="pagination-controls"></div>

        <div id="subpanel-container" style="display:none; flex-direction:column; flex:1; overflow:hidden;">
            <div id="subpanel-header" style="padding: 15px; border-bottom: 1px solid var(--cyan); display:flex; justify-content:space-between; align-items:center; background: rgba(0,0,0,0.5);">
                <span id="subpanel-title" style="color:var(--cyan); font-family:var(--font-ui); font-weight:bold; font-size:0.8rem;">TITULO</span>
                <button onclick="closeSubPanel()" style="background:transparent; border:none; color:var(--red); cursor:pointer; font-weight:bold; font-family:var(--font-mono); font-size:0.8rem;">[ VOLVER ]</button>
            </div>
            <div id="subpanel-list" class="list-container"></div>
        </div>

        <div class="ai-monitor" id="log">
            </div>
    </div>
</div>

<div id="mobile-floating-node" onclick="scrollToActiveCard()">
    <div class="node-badge"><i class="pulse-dot"></i> NODO ACTUAL</div>
    <div id="mobile-floating-name">ESPERANDO...</div>
</div>

<script>
    // ==========================================
    // INICIALIZACIÓN DE FIREBASE (NUBE)
    // ==========================================
    const firebaseConfig = {
      apiKey: "AIzaSyB95MjOlPdX2zxrez03tRM_vtdgo2GD7fY",
      authDomain: "ventas-online-48fa8.firebaseapp.com",
      databaseURL: "https://ventas-online-48fa8-default-rtdb.firebaseio.com",
      projectId: "ventas-online-48fa8",
      storageBucket: "ventas-online-48fa8.firebasestorage.app",
      messagingSenderId: "429220607663",
      appId: "1:429220607663:web:012548d569c24439d4c65a"
    };
    
    // Iniciar Firebase
    if (!firebase.apps.length) {
        firebase.initializeApp(firebaseConfig);
    }
    const database = firebase.database();

    // ==========================================
    // LÓGICA DE RANKING Y REACCIONES (HIVE SYNC)
    // ==========================================
    let globalLikesDB = JSON.parse(localStorage.getItem('CORTEX_GLOBAL_LIKES')) || { resetTime: Date.now(), likes: {} };
    const SEVEN_DAYS_MS = 7 * 24 * 60 * 60 * 1000;

    // Resetear ranking cada 7 días
    if (Date.now() - globalLikesDB.resetTime > SEVEN_DAYS_MS) {
        globalLikesDB = { resetTime: Date.now(), likes: {} };
        localStorage.setItem('CORTEX_GLOBAL_LIKES', JSON.stringify(globalLikesDB));
    }

    function sendReaction(emoji) {
        if(currentPlayingIdx === -1) return;
        let canal = DB[currentPlayingIdx];
        
        createFloatingEmoji(emoji);

        if(emoji === '❤️') {
            if (!globalLikesDB.likes[canal.url]) globalLikesDB.likes[canal.url] = 0;
            globalLikesDB.likes[canal.url]++;
            localStorage.setItem('CORTEX_GLOBAL_LIKES', JSON.stringify(globalLikesDB));
            logger(`+1 ME GUSTA REGISTRADO: ${canal.name}`, "var(--red)");
            database.ref('CORTEX/likes').child(hashString(canal.url)).set(globalLikesDB.likes[canal.url]);
        }

        if(currentUser) {
            database.ref('CORTEX/global_events').push({
                emoji: emoji, channelUrl: canal.url, timestamp: Date.now()
            });
        }
    }

    function createFloatingEmoji(emoji) {
        const viewport = document.getElementById('reaction-viewport');
        if(!viewport) return;
        const el = document.createElement('div');
        el.className = 'floating-reaction';
        el.innerText = emoji;
        el.style.left = (Math.random() * 60 + 20) + '%';
        viewport.appendChild(el);
        setTimeout(() => el.remove(), 2000);
    }

    database.ref('CORTEX/global_events').limitToLast(1).on('child_added', (snap) => {
        const data = snap.val();
        if(data && Date.now() - data.timestamp < 5000 && currentPlayingIdx !== -1) {
            if(DB[currentPlayingIdx] && DB[currentPlayingIdx].url === data.channelUrl) {
                createFloatingEmoji(data.emoji);
            }
        }
    });

    window.addEventListener('error', (e) => {
        logger(`[IA ARREGLADOR] ERROR PREVENIDO: ${e.message}`, "var(--red)");
        setAIAgentState(true, "COLAPSO PREVENIDO // ESTABILIZANDO...");
        setTimeout(() => setAIAgentState(false), 2000);
    });
    window.addEventListener('unhandledrejection', (e) => {
        logger(`[IA ARREGLADOR] FALLO DE RED SILENCIADO. SISTEMA ESTABLE.`, "var(--amber)");
    });

    const DB_NAME = "CORTEX_GLOBAL_MATRIX";
    let cortexLocalDB;

    let currentPage = 1;
    const itemsPerPage = 250;

    function initMatrixDB() {
        return new Promise((resolve, reject) => {
            const request = indexedDB.open(DB_NAME, 1);
            request.onupgradeneeded = (event) => {
                cortexLocalDB = event.target.result;
                if (!cortexLocalDB.objectStoreNames.contains('nodes')) {
                    cortexLocalDB.createObjectStore('nodes', { keyPath: 'region' });
                }
            };
            request.onsuccess = (event) => { cortexLocalDB = event.target.result; resolve(); };
            request.onerror = (event) => reject(event.target.error);
        });
    }

    function saveMatrixToLocal(region, data) {
        if (!cortexLocalDB) return;
        const tx = cortexLocalDB.transaction('nodes', 'readwrite');
        const store = tx.objectStore('nodes');
        store.put({ region: region, data: data, lastUpdate: Date.now() });
    }

    function getMatrixFromLocal(region) {
        return new Promise((resolve, reject) => {
            if (!cortexLocalDB) return resolve(null);
            const tx = cortexLocalDB.transaction('nodes', 'readonly');
            const store = tx.objectStore('nodes');
            const request = store.get(region);
            request.onsuccess = () => resolve(request.result ? request.result.data : null);
            request.onerror = () => resolve(null);
        });
    }

    let inactivityTimer = null;
    function resetInactivity() {
        const sidebarEl = document.querySelector('.sidebar');
        const osdEl = document.getElementById('ui-osd');
        const bottomBarEl = document.getElementById('bottom-bar');
        const emojiBubble = document.getElementById('emoji-bubble');
        const maxBtn = document.getElementById('floating-max-btn');
        
        if(osdEl) osdEl.classList.remove('hidden-osd');
        if(bottomBarEl) bottomBarEl.classList.remove('hidden-bottom-bar');
        if(emojiBubble && currentPlayingIdx !== -1) emojiBubble.classList.remove('hidden-bubble');
        if(maxBtn) maxBtn.classList.remove('hidden-max-btn');
        
        if(window.innerWidth > 900) { if(sidebarEl) sidebarEl.classList.remove('hidden-sidebar'); }
        
        if(inactivityTimer) clearTimeout(inactivityTimer);
        
        inactivityTimer = setTimeout(() => {
            if(osdEl) osdEl.classList.add('hidden-osd');
            if(bottomBarEl) bottomBarEl.classList.add('hidden-bottom-bar');
            if(emojiBubble) emojiBubble.classList.add('hidden-bubble');
            if(maxBtn) maxBtn.classList.add('hidden-max-btn');
            if(window.innerWidth > 900) { if(sidebarEl) sidebarEl.classList.add('hidden-sidebar'); }
        }, 10000); 
    }

    window.addEventListener('mousemove', resetInactivity);
    window.addEventListener('mousedown', resetInactivity);
    window.addEventListener('keydown', resetInactivity);
    window.addEventListener('touchstart', resetInactivity);

    window.addEventListener('click', (e) => {
        if(e.target.id !== 'in-player-search') {
            const res = document.getElementById('zapper-results');
            if(res) res.style.display = 'none';
        }
    });

    let DB = [];
    let hls = null;
    let currentPlayingIdx = -1;
    let currentChannelName = ""; 
    let aiRecoveryTimeout = null;
    let currentSubPanel = ""; 
    let zapperMatches = []; 

    const videoCore = document.getElementById('video-core');
    const videoAV1 = document.getElementById('video-av1-panel');
    const playerBox = document.getElementById('main-player-box');
    const osdRes = document.getElementById('osd-res');
    const adminTelemetryPanel = document.getElementById('admin-telemetry-panel');
    
    let usersDB = JSON.parse(localStorage.getItem('CORTEX_USERS')) || [];
    let currentUser = null; 
    let monitorInterval = null; 

    database.ref('CORTEX/usersDB').on('value', (snapshot) => {
        const data = snapshot.val();
        if (data) {
            usersDB = data;
            localStorage.setItem('CORTEX_USERS', JSON.stringify(usersDB));
            if (currentUser && currentUser.role === 'admin') {
                renderAdminPanel(); logger("DATOS DE LA COLMENA SINCRONIZADOS DESDE LA NUBE", "var(--cyan)");
            }
        } else {
            if(!usersDB.find(u => u.email === 'tv@tv.com')) {
                usersDB.push({
                    email: 'tv@tv.com', pass: 'adm4409405', keyword: 'teamo', role: 'admin', status: 'approved', 
                    name: 'OVERLORD', photo: '', favorites: [], savedStable: [], workingNodes: []
                });
                saveUsers();
            }
        }
    });

    function saveUsers() { 
        localStorage.setItem('CORTEX_USERS', JSON.stringify(usersDB));
        database.ref('CORTEX/usersDB').set(usersDB); 
    }

    let chatDB = JSON.parse(localStorage.getItem('CORTEX_CHAT')) || {};
    let errorDB = JSON.parse(localStorage.getItem('CORTEX_ERRORS')) || [];
    let aiStableDB = JSON.parse(localStorage.getItem('CORTEX_AI_STABLE_DB')) || []; 
    let liveDB = JSON.parse(localStorage.getItem('CORTEX_LIVE_DB')) || [];
    let deadDB = JSON.parse(localStorage.getItem('CORTEX_DEAD_DB')) || [];
    let iaRepairDB = JSON.parse(localStorage.getItem('CORTEX_IA_REPAIR_DB')) || [];

    function toggleFullScreen() {
        const player = document.getElementById('main-player-box');
        if (!document.fullscreenElement && !document.webkitFullscreenElement && !document.msFullscreenElement) {
            if (player.requestFullscreen) { player.requestFullscreen().then(() => lockOrientation()).catch(err => console.log(err)); } 
            else if (player.webkitRequestFullscreen) { player.webkitRequestFullscreen(); lockOrientation(); } 
            else if (player.msRequestFullscreen) { player.msRequestFullscreen(); lockOrientation(); }
        } else {
            if (document.exitFullscreen) { document.exitFullscreen(); } 
            else if (document.webkitExitFullscreen) { document.webkitExitFullscreen(); } 
            else if (document.msExitFullscreen) { document.msExitFullscreen(); }
            unlockOrientation();
        }
    }

    function lockOrientation() {
        if (screen.orientation && screen.orientation.lock) {
            screen.orientation.lock('landscape').catch(function(error) { console.log("No se pudo forzar la rotación horizontal automáticamente: ", error); });
        }
    }

    function unlockOrientation() {
        if (screen.orientation && screen.orientation.unlock) { screen.orientation.unlock(); }
    }

    document.addEventListener('fullscreenchange', handleFullscreenChange);
    document.addEventListener('webkitfullscreenchange', handleFullscreenChange);
    document.addEventListener('mozfullscreenchange', handleFullscreenChange);
    document.addEventListener('MSFullscreenChange', handleFullscreenChange);

    function handleFullscreenChange() {
        if (!document.fullscreenElement && !document.webkitIsFullScreen && !document.mozFullScreen && !document.msFullscreenElement) { unlockOrientation(); }
    }

    function handleInPlayerSearch() {
        const query = document.getElementById('in-player-search').value.toLowerCase();
        const resultsContainer = document.getElementById('zapper-results');
        if(query.length < 1) { resultsContainer.style.display = 'none'; zapperMatches = []; return; }

        zapperMatches = DB.map((c, i) => ({...c, originalIdx: i}))
                          .filter(c => c.name.toLowerCase().includes(query) || String(c.number).toLowerCase().includes(query))
                          .slice(0, 15); 

        if(zapperMatches.length > 0) {
            resultsContainer.innerHTML = zapperMatches.map(c => `
                <div class="zapper-item" onclick="playFromZapper(${c.originalIdx})">
                    <span style="color:var(--cyan); font-weight:bold;">[CAM-${c.number}]</span> ${c.name}
                </div>
            `).join('');
            resultsContainer.style.display = 'block';
        } else {
            resultsContainer.innerHTML = '<div style="padding: 8px; color: var(--red); font-size: 0.7rem; text-align:center;">NO HAY COINCIDENCIAS</div>';
            resultsContainer.style.display = 'block';
        }
    }

    function playFromZapper(idx) {
        document.getElementById('in-player-search').value = '';
        document.getElementById('zapper-results').style.display = 'none';
        play(DB[idx], idx);
    }

    function handleInPlayerSearchKey(e) {
        if(e.key === 'Enter') { if(zapperMatches.length > 0) { playFromZapper(zapperMatches[0].originalIdx); } }
    }

    // --- FUNCIONES DEL NUEVO MENÚ OSD ---
    function toggleOSDMenu() {
        const menu = document.getElementById('osd-tech-menu');
        menu.style.display = (menu.style.display === 'block') ? 'none' : 'block';
        resetInactivity();
    }

    function updateOSDSettings() {
        resetInactivity();
        const vol = document.getElementById('osd-vol').value;
        const bri = document.getElementById('osd-bri').value;
        const con = document.getElementById('osd-con').value;
        
        videoCore.volume = vol;
        videoAV1.volume = vol;
        
        videoCore.style.filter = `brightness(${bri}) contrast(${con})`;
        videoAV1.style.filter = `brightness(${bri}) contrast(${con})`;
    }

    // Telemetría simulada en tiempo real para el OSD
    setInterval(() => {
        const menu = document.getElementById('osd-tech-menu');
        if(menu && menu.style.display === 'block') {
            // Generar fluctuación realista del buffer
            const bufferHealth = Math.floor(Math.random() * (100 - 85 + 1)) + 85; 
            const bufferStat = document.getElementById('stat-buffer');
            if(bufferStat) {
                bufferStat.innerText = `${bufferHealth}%`;
                bufferStat.style.color = bufferHealth < 90 ? 'var(--amber)' : '#fff';
            }
        }
    }, 2000);

    const WORLD_REGIONS = [
        { code: 'index', name: '🌐 RED GLOBAL (MUNDIAL)', flag: '🌍' },
        { code: 'index_nsfw', name: '🔞 RED GLOBAL (+18)', flag: '🔞' },
        { code: 'ar', name: 'Argentina', flag: '🇦🇷' },
        { code: 'br', name: 'Brasil', flag: '🇧🇷' },
        { code: 'cl', name: 'Chile', flag: '🇨🇱' },
        { code: 'co', name: 'Colombia', flag: '🇨🇴' },
        { code: 'es', name: 'España', flag: '🇪🇸' },
        { code: 'mx', name: 'México', flag: '🇲🇽' },
        { code: 'us', name: 'Estados Unidos', flag: '🇺🇸' },
        { code: 'pe', name: 'Perú', flag: '🇵🇪' },
        { code: 've', name: 'Venezuela', flag: '🇻🇪' },
        { code: 'cat_sports', name: 'Deportes', flag: '⚽' },
        { code: 'cat_movies', name: 'Cine', flag: '🎬' },
        { code: 'cat_news', name: 'Noticias', flag: '📰' }
    ];

    function openWorldDirectory() {
        currentSubPanel = 'world';
        document.getElementById('list').style.display = 'none';
        document.getElementById('pagination-controls').style.display = 'none'; 
        document.getElementById('main-controls').style.display = 'none';
        document.getElementById('user-personal-panels').style.display = 'none';
        if(document.getElementById('admin-tools').style.display === 'block') {
            document.getElementById('admin-tools').style.display = 'none';
            document.getElementById('admin-tools').classList.add('was-open');
        }

        document.getElementById('subpanel-container').style.display = 'flex';
        document.getElementById('subpanel-title').innerText = "🌍 DIRECTORIO MUNDIAL";
        document.getElementById('subpanel-title').style.color = "var(--cyan)";
        document.getElementById('subpanel-header').style.borderColor = "var(--cyan)";

        const subList = document.getElementById('subpanel-list');
        subList.innerHTML = WORLD_REGIONS.map(r => `
            <div class="card" onclick="document.getElementById('region').value='${r.code}'; changeRegion('${r.code}'); closeSubPanel();" style="border-color: rgba(0,243,255,0.2);">
                <div style="font-size:1.5rem; margin-right:15px;">${r.flag}</div>
                <div style="flex:1;">
                    <div style="color:var(--cyan); font-family:var(--font-ui); font-weight:bold; font-size:0.9rem;">${r.name}</div>
                    <div style="font-size:0.6rem; color:#888;">[ INICIAR ENLACE DE RED ]</div>
                </div>
            </div>
        `).join('');
    }

    function openSubPanel(type) {
        currentSubPanel = type;
        document.getElementById('list').style.display = 'none';
        document.getElementById('pagination-controls').style.display = 'none'; 
        document.getElementById('main-controls').style.display = 'none';
        document.getElementById('user-personal-panels').style.display = 'none';
        
        if(document.getElementById('admin-tools').style.display === 'block') {
            document.getElementById('admin-tools').style.display = 'none';
            document.getElementById('admin-tools').classList.add('was-open');
        }

        document.getElementById('subpanel-container').style.display = 'flex';
        
        const subList = document.getElementById('subpanel-list');
        let dataArr = []; let title = ""; let color = "";

        if (type === 'ranking') {
            dataArr = DB.filter(c => globalLikesDB.likes[c.url]).map(c => ({...c, likeCount: globalLikesDB.likes[c.url]})).sort((a, b) => b.likeCount - a.likeCount).slice(0, 50);
            title = "🏆 TOP CANALES (RANKING SEMANAL)"; color = "var(--gold)";
        } else if(type === 'favs') {
            dataArr = currentUser.favorites || []; title = "⭐ MIS NODOS PREFERIDOS"; color = "var(--amber)";
        } else if(type === 'saved') {
            dataArr = currentUser.savedStable || []; title = "💾 MIS SEÑALES GUARDADAS"; color = "var(--green)";
        } else if(type === 'auto') {
            dataArr = currentUser.workingNodes || []; title = "📡 SEÑALES ACTIVAS (AUTO)"; color = "var(--cyan)";
        }

        document.getElementById('subpanel-title').innerText = title; document.getElementById('subpanel-title').style.color = color;
        document.getElementById('subpanel-header').style.borderColor = color;

        if (type === 'ranking') renderRankingIntoContainer(dataArr, subList); else renderListIntoContainer(dataArr, subList, type);
    }

    function renderRankingIntoContainer(dataArr, container) {
        const fragment = document.createDocumentFragment();
        if(dataArr.length === 0) { container.innerHTML = '<div style="text-align:center; padding: 20px; color: #888;">SIN DATOS DE RANKING</div>'; return; }
        dataArr.forEach((c, index) => {
            const card = document.createElement('div');
            const rankClass = index < 3 ? `rank-${index}` : '';
            card.className = `card ${rankClass}`;
            const defaultLogo = "https://via.placeholder.com/40/000000/00f3ff?text=CAM";
            card.innerHTML = `
                <div style="font-family: var(--font-ui); font-size: 1.1rem; color: var(--gold); margin-right: 15px; font-weight:900; min-width:35px;">#${index + 1}</div>
                <img src="${c.logo || defaultLogo}" style="width: 30px; height: 30px; border-radius: 4px; margin-right: 10px; object-fit: contain; background: #000;">
                <div style="flex:1; overflow: hidden;">
                    <div style="font-size:0.95rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">${c.name}</div>
                    <div style="font-size:0.65rem; color:var(--red); font-weight:bold;">❤️ ${c.likeCount} LIKES</div>
                </div>`;
            card.onclick = async () => {
                if(c.region && c.region !== document.getElementById('region').value) {
                    logger(`Sincronizando categoría a: ${c.region.toUpperCase()}`, "var(--cyan)");
                    document.getElementById('region').value = c.region; localStorage.setItem('CORTEX_REGION', c.region); await loadNodes(c.region); 
                }
                const originalIdx = DB.findIndex(x => x.url === c.url);
                play(originalIdx !== -1 ? DB[originalIdx] : c, originalIdx !== -1 ? originalIdx : -1);
            };
            fragment.appendChild(card);
        });
        container.innerHTML = ''; container.appendChild(fragment);
    }

    function closeSubPanel() {
        currentSubPanel = ""; document.getElementById('subpanel-container').style.display = 'none';
        document.getElementById('list').style.display = 'block'; document.getElementById('pagination-controls').style.display = 'flex'; 
        document.getElementById('main-controls').style.display = 'block'; document.getElementById('user-personal-panels').style.display = 'flex';
        if(document.getElementById('admin-tools').classList.contains('was-open')) {
            document.getElementById('admin-tools').style.display = 'block'; document.getElementById('admin-tools').classList.remove('was-open');
        }
    }

    function renderListIntoContainer(dataArr, container, type) {
        const fragment = document.createDocumentFragment();
        if(dataArr.length === 0) { container.innerHTML = '<div style="text-align:center; padding: 20px; color: #888;">VACÍO</div>'; return; }
        dataArr.forEach((c) => {
            const card = document.createElement('div'); card.className = `card`; 
            const defaultLogo = "https://via.placeholder.com/40/000000/00f3ff?text=CAM";
            const displayLogo = c.logo || defaultLogo; const displayNumber = `CAM-${c.number}`;
            card.innerHTML = `
                <div style="font-family: var(--font-ui); font-size: 0.7rem; color: var(--cyan); margin-right: 10px; opacity: 0.7; flex-shrink: 0; font-weight:bold; min-width: 60px;">[${displayNumber}]</div>
                <img src="${displayLogo}" style="width: 30px; height: 30px; border-radius: 4px; border: 1px solid rgba(0,243,255,0.3); margin-right: 10px; object-fit: contain; background: #000; flex-shrink: 0;" onerror="this.src='${defaultLogo}'">
                <div class="status-dot st-${c.status}"></div>
                <div style="flex:1; overflow: hidden;">
                    <div style="font-size:0.95rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">${c.name}</div>
                    <div style="font-size:0.65rem; color:#888; margin-top: 3px;">ESTADO: <span style="color: ${c.status === 'live' ? 'var(--green)' : c.status === 'dead' ? 'var(--red)' : 'var(--amber)'}">${c.status.toUpperCase()}</span></div>
                </div>
                <button onclick="removeUserChannel('${c.url}', '${type}', event)" style="background:rgba(255,0,85,0.1); border:1px solid var(--red); color:var(--red); padding:3px 6px; border-radius:3px; cursor:pointer; font-size:0.6rem; font-weight:bold; margin-left:10px; z-index:20;">X</button>
            `;
            card.onclick = async () => {
                if(c.region && c.region !== document.getElementById('region').value) {
                    logger(`Sincronizando categoría a: ${c.region.toUpperCase()}`, "var(--cyan)");
                    document.getElementById('region').value = c.region; localStorage.setItem('CORTEX_REGION', c.region); await loadNodes(c.region); 
                }
                const originalIdx = DB.findIndex(x => x.url === c.url); play(originalIdx !== -1 ? DB[originalIdx] : c, originalIdx !== -1 ? originalIdx : -1);
            };
            fragment.appendChild(card);
        });
        container.innerHTML = ''; container.appendChild(fragment);
    }

    function removeUserChannel(url, type, event) {
        event.stopPropagation(); if(!currentUser) return;
        if(type === 'favs') currentUser.favorites = currentUser.favorites.filter(c => c.url !== url);
        else if(type === 'saved') currentUser.savedStable = currentUser.savedStable.filter(c => c.url !== url);
        else if(type === 'auto') currentUser.workingNodes = currentUser.workingNodes.filter(c => c.url !== url);
        saveUsers(); openSubPanel(type); 
    }

    function addCurrentToFavs() {
        if(!currentUser || currentPlayingIdx === -1) return;
        let canal = {...DB[currentPlayingIdx], region: document.getElementById('region').value};
        if(!currentUser.favorites) currentUser.favorites = [];
        if(!currentUser.favorites.find(c => c.url === canal.url)) {
            currentUser.favorites.push(canal); saveUsers(); if(currentSubPanel === 'favs') openSubPanel('favs'); logger(`AÑADIDO A PREFERIDOS: ${canal.name}`, "var(--amber)");
        }
    }

    function addCurrentToSaved() {
        if(!currentUser || currentPlayingIdx === -1) return;
        let canal = {...DB[currentPlayingIdx], region: document.getElementById('region').value};
        if(!currentUser.savedStable) currentUser.savedStable = [];
        if(!currentUser.savedStable.find(c => c.url === canal.url)) {
            currentUser.savedStable.push(canal); saveUsers(); if(currentSubPanel === 'saved') openSubPanel('saved'); logger(`SEÑAL ESTABLE GUARDADA: ${canal.name}`, "var(--green)");
        }
    }

    function runUserAIspection() {
        if (!currentUser) return;
        logger("INICIANDO INSPECCIÓN IA DE NODOS...", "var(--purple)"); setAIAgentState(true, "INSPECCIONANDO MATRIZ DE RED...");
        setTimeout(() => {
            let knownLive = DB.filter(c => c.status === 'live'); let potentialNodes = DB.filter(c => c.status !== 'dead' && !c.name.includes("DESCONOCIDO")).slice(0, 150);
            let added = 0; let listToInspect = knownLive.length > 0 ? knownLive : potentialNodes; let scanBatch = listToInspect.sort(() => 0.5 - Math.random()).slice(0, 15);
            if (!currentUser.workingNodes) currentUser.workingNodes = [];
            scanBatch.forEach(node => {
                let formattedNode = {...node, region: document.getElementById('region').value};
                if (!currentUser.workingNodes.find(n => n.url === formattedNode.url)) { currentUser.workingNodes.push(formattedNode); added++; }
            });
            saveUsers(); setAIAgentState(false);
            if (added > 0) {
                logger(`[IA] INSPECCIÓN COMPLETADA. ${added} SEÑALES ACTIVAS GUARDADAS.`, "var(--green)");
                alert(`¡Inspección IA Completada!\nSe han detectado y guardado ${added} señales estables en tu panel de Señales Activas.`);
                if(currentSubPanel === 'auto') openSubPanel('auto');
            } else {
                logger(`[IA] NO SE ENCONTRARON SEÑALES NUEVAS EN ESTA ZONA.`, "var(--amber)"); alert("La IA ha revisado la zona actual y todos los nodos estables disponibles ya están en tu lista.");
            }
        }, 2500); 
    }

    function scrollToActiveCard() {
        if(currentPlayingIdx === -1) return;
        if(currentSubPanel === 'world') closeSubPanel();
        const filter = document.getElementById('search').value.toLowerCase();
        let mappedDB = DB.map((c, originalIdx) => ({ ...c, originalIdx }));
        if (filter) { mappedDB = mappedDB.filter(c => c.name.toLowerCase().includes(filter)); }
        const positionInFiltered = mappedDB.findIndex(c => c.originalIdx === currentPlayingIdx);
        if (positionInFiltered !== -1) {
            const correctPage = Math.floor(positionInFiltered / itemsPerPage) + 1;
            if (currentPage !== correctPage && currentSubPanel === "") { currentPage = correctPage; render(); }
        }
        setTimeout(() => {
            const activeCard = document.querySelector('.card.active');
            if(activeCard) {
                activeCard.scrollIntoView({ behavior: "smooth", block: "center" });
                activeCard.style.transition = "box-shadow 0.3s"; activeCard.style.boxShadow = "inset 0 0 20px var(--cyan)";
                setTimeout(() => activeCard.style.boxShadow = "none", 1500);
            }
        }, 200);
    }

    function playNextChannel() {
        if(!currentUser) return;
        if (currentSubPanel === 'favs' || currentSubPanel === 'saved' || currentSubPanel === 'auto') {
            let list = [];
            if(currentSubPanel === 'favs') list = currentUser.favorites || [];
            else if(currentSubPanel === 'saved') list = currentUser.savedStable || [];
            else if(currentSubPanel === 'auto') list = currentUser.workingNodes || [];
            if (list.length === 0) return;
            let idxInList = list.findIndex(c => c.url === DB[currentPlayingIdx]?.url);
            if(idxInList === -1) idxInList = 0; else { idxInList++; if (idxInList >= list.length) idxInList = 0; }
            let nextCanal = list[idxInList]; let dbIdx = DB.findIndex(x => x.url === nextCanal.url);
            play(nextCanal, dbIdx !== -1 ? dbIdx : -1); return;
        }
        if(DB.length === 0) return; let nextIdx = currentPlayingIdx + 1; if(nextIdx >= DB.length) nextIdx = 0; 
        let startIdx = nextIdx; while(DB[nextIdx].status === 'dead') { 
            nextIdx++; if(nextIdx >= DB.length) nextIdx = 0; if(nextIdx === startIdx) { logger("TODOS LOS CANALES CAÍDOS", "var(--red)"); return; }
        }
        play(DB[nextIdx], nextIdx); 
    }

    function playPrevChannel() {
        if(!currentUser) return;
        if (currentSubPanel === 'favs' || currentSubPanel === 'saved' || currentSubPanel === 'auto') {
            let list = [];
            if(currentSubPanel === 'favs') list = currentUser.favorites || [];
            else if(currentSubPanel === 'saved') list = currentUser.savedStable || [];
            else if(currentSubPanel === 'auto') list = currentUser.workingNodes || [];
            if (list.length === 0) return;
            let idxInList = list.findIndex(c => c.url === DB[currentPlayingIdx]?.url);
            if(idxInList === -1) idxInList = list.length - 1; else { idxInList--; if (idxInList < 0) idxInList = list.length - 1; }
            let prevCanal = list[idxInList]; let dbIdx = DB.findIndex(x => x.url === prevCanal.url);
            play(prevCanal, dbIdx !== -1 ? dbIdx : -1); return;
        }
        if(DB.length === 0) return; let prevIdx = currentPlayingIdx - 1; if(prevIdx < 0) prevIdx = DB.length - 1; 
        let startIdx = prevIdx; while(DB[prevIdx].status === 'dead') { 
            prevIdx--; if(prevIdx < 0) prevIdx = DB.length - 1; if(prevIdx === startIdx) { logger("TODOS LOS CANALES CAÍDOS", "var(--red)"); return; }
        }
        play(DB[prevIdx], prevIdx);
    }

    function showAdminTelemetry(canalName, isError = false, errorMessage = "") {
        if(currentUser && currentUser.role === 'admin') {
            adminTelemetryPanel.style.display = 'block'; document.getElementById('telemetry-channel').innerText = `CÁMARA: ${canalName}`;
            if(isError) {
                document.getElementById('telemetry-status').innerHTML = `ESTADO: <span style="color:var(--red);">FALLO DETECTADO</span>`; document.getElementById('telemetry-error').innerText = `RAZÓN: ${errorMessage}`;
            } else {
                document.getElementById('telemetry-status').innerHTML = `ESTADO: <span style="color:var(--green);">TRANSMITIENDO / ESTABLE</span>`; document.getElementById('telemetry-error').innerText = ``;
            }
        }
    }
    
    function hideAdminTelemetry() { adminTelemetryPanel.style.display = 'none'; }

    function setAIAgentState(isActive, message = "") {
        const bubble = document.getElementById('ai-bubble'); const userPanel = document.getElementById('ai-user-panel');
        const bubbleMsg = document.getElementById('ai-bubble-msg'); const panelMsg = document.getElementById('ai-user-panel-msg');
        if(isActive) {
            bubble.style.display = 'flex'; userPanel.style.display = 'block'; bubbleMsg.innerText = message; panelMsg.innerText = message;
        } else { bubble.style.display = 'none'; userPanel.style.display = 'none'; }
    }

    function updateVideoTelemetry(vidElement) {
        if(vidElement.videoWidth && vidElement.videoHeight) {
            osdRes.innerText = `RESOLUCIÓN: ${vidElement.videoWidth}x${vidElement.videoHeight}px`; osdRes.style.display = 'block';
        } else { osdRes.style.display = 'none'; }
    }
    videoCore.addEventListener('resize', () => updateVideoTelemetry(videoCore)); videoCore.addEventListener('loadedmetadata', () => updateVideoTelemetry(videoCore));
    videoAV1.addEventListener('resize', () => updateVideoTelemetry(videoAV1)); videoAV1.addEventListener('loadedmetadata', () => updateVideoTelemetry(videoAV1));

    window.onload = async () => { 
        logger("INICIANDO SECUENCIA DE ARRANQUE..."); resetInactivity(); 
        await initMatrixDB().catch(() => logger("ERROR INICIANDO BD LOCAL", "var(--red)"));
        const savedRegion = localStorage.getItem('CORTEX_REGION') || 'co'; document.getElementById('region').value = savedRegion;
        const session = localStorage.getItem('CORTEX_SESSION'); if(session) { const u = usersDB.find(x => x.email === session); if(u) setSession(u); }
        loadNodes(savedRegion); 
    };

    function logChannelErrorAndFix(canalNombre, canalUrl, tipoError, reasonDetails) {
        const time = new Date().toLocaleString(); errorDB.push({ name: canalNombre, url: canalUrl, type: tipoError, time: time }); localStorage.setItem('CORTEX_ERRORS', JSON.stringify(errorDB));
        const repairEntry = { name: canalNombre, url: canalUrl, reason: reasonDetails, time: time };
        if(!iaRepairDB.find(x => x.name === canalNombre)) { iaRepairDB.push(repairEntry); localStorage.setItem('CORTEX_IA_REPAIR_DB', JSON.stringify(iaRepairDB)); }
    }

    function syncSignalDatabases() {
        liveDB = DB.filter(c => c.status === 'live'); deadDB = DB.filter(c => c.status === 'dead');
        localStorage.setItem('CORTEX_LIVE_DB', JSON.stringify(liveDB)); localStorage.setItem('CORTEX_DEAD_DB', JSON.stringify(deadDB));
        if(currentUser && currentUser.role === 'admin') { renderSignalPanels(); }
    }

    function renderSignalPanels() {
        const pLive = document.getElementById('panel-live'); const pDead = document.getElementById('panel-dead'); if(!pLive || !pDead) return;
        pLive.innerHTML = liveDB.map(c => `<div style="white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">✓ ${c.name}</div>`).join('');
        pDead.innerHTML = deadDB.map(c => `<div style="white-space:nowrap; overflow:hidden; text-overflow:ellipsis;">✗ ${c.name}</div>`).join('');
    }

    function runAiSearch() {
        logger("INICIANDO BUSCADOR IA DE NUEVAS SEÑALES...", "var(--cyan)");
        setTimeout(() => {
            const nuevasSenales = DB.filter(c => c.status !== 'dead' && Math.random() > 0.5); 
            nuevasSenales.forEach(s => { if(!aiStableDB.find(x => x.url === s.url)) { aiStableDB.push(s); } });
            localStorage.setItem('CORTEX_AI_STABLE_DB', JSON.stringify(aiStableDB));
            logger(`RASTREO COMPLETADO: ${nuevasSenales.length} SEÑALES SINCRONIZADAS EN BD.`, "var(--green)");
            alert(`El Buscador IA ha encontrado y sincronizado ${nuevasSenales.length} canales estables en la BD Inteligente.`);
        }, 2000);
    }

    function toggleAiMonitor() {
        const btn = document.getElementById('btn-monitor-ia');
        if(monitorInterval) {
            clearInterval(monitorInterval); monitorInterval = null; btn.innerText = "ACTIVAR SUPERVISOR IA EN TIEMPO REAL"; btn.style.background = "rgba(10, 255, 0, 0.2)"; logger("SUPERVISOR IA DESACTIVADO.", "var(--amber)");
        } else {
            btn.innerText = "DESACTIVAR SUPERVISOR IA"; btn.style.background = "rgba(255, 174, 0, 0.2)"; logger("SUPERVISOR IA ACTIVADO EN SEGUNDO PLANO.", "var(--green)");
            monitorInterval = setInterval(() => { syncSignalDatabases(); if(currentPlayingIdx !== -1 && hls) { logger(`[IA SYNC] ACTUALIZANDO BD DE PANELES EN TIEMPO REAL...`, "var(--cyan)"); } }, 5000); 
        }
    }

    function openLogin() {
        if(currentUser) { currentUser = null; localStorage.removeItem('CORTEX_SESSION'); location.reload(); return; }
        document.getElementById('login-modal').style.display = 'flex';
    }
    function closeLogin() { document.getElementById('login-modal').style.display = 'none'; }

    function registerUser() {
        const email = document.getElementById('auth-email').value; const pass = document.getElementById('auth-pass').value; const keyword = document.getElementById('auth-keyword').value;
        if(!email || !pass || !keyword) return alert("LLENA TODOS LOS DATOS INCLUYENDO LA PALABRA CLAVE");
        if(usersDB.find(u => u.email === email)) return alert("ESTE USUARIO YA EXISTE");
        usersDB.push({ email: email, pass: pass, keyword: keyword.toLowerCase().trim(), role: 'user', status: 'pending', daysLeft: 7, lastDate: '', viewedChannels: [], favorites: [], savedStable: [], workingNodes: [], name: 'OPERADOR NUEVO', photo: '' });
        saveUsers(); logger(`NUEVO REGISTRO ENVIADO: ${email}`, 'var(--cyan)'); alert("REGISTRO EXITOSO. ESPERA LA AUTORIZACIÓN DEL ADMINISTRADOR."); closeLogin();
    }

    function loginUser() {
        const email = document.getElementById('auth-email').value; const pass = document.getElementById('auth-pass').value;
        const u = usersDB.find(user => user.email === email && user.pass === pass);
        if(!u) return alert("ACCESO DENEGADO. CÓDIGO INCORRECTO.");
        if(u.status === 'blocked') return alert("CUENTA BLOQUEADA POR EL ADMINISTRADOR.");
        if(u.status !== 'approved' && u.role !== 'admin') return alert("CUENTA NO APROBADA AÚN.");
        setSession(u); closeLogin();
    }

    function recoverPassword() {
        const email = document.getElementById('auth-email').value; const keyword = document.getElementById('auth-keyword').value;
        if(!email || !keyword) return alert("INGRESA TU IDENTIFICACIÓN (CORREO) Y PALABRA CLAVE PARA RECUPERAR.");
        const u = usersDB.find(user => user.email === email && user.keyword === keyword.toLowerCase().trim());
        if(u) { alert(`SISTEMA DE RECUPERACIÓN:\nTu código de seguridad es: ${u.pass}`); logger(`CÓDIGO RECUPERADO PARA: ${email}`, 'var(--amber)'); } 
        else { alert("DATOS INCORRECTOS. NO SE ENCONTRÓ COINCIDENCIA DE PALABRA CLAVE."); }
    }

    function setSession(u) {
        currentUser = u; localStorage.setItem('CORTEX_SESSION', u.email);
        if(!currentUser.favorites) currentUser.favorites = []; if(!currentUser.savedStable) currentUser.savedStable = []; if(!currentUser.workingNodes) currentUser.workingNodes = [];
        document.getElementById('btn-open-login').innerText = "DESCONECTAR"; document.getElementById('btn-profile').style.display = "inline-block"; document.getElementById('user-personal-panels').style.display = "flex";
        if(u.role === 'admin') {
            document.getElementById('admin-tools').style.display = 'block'; document.getElementById('user-timer').style.display = 'none';
            document.getElementById('osd-name').innerText = "MODO ADMINISTRADOR"; document.getElementById('osd-name').style.color = "var(--amber)"; document.getElementById('ai-count').innerText = `ACCESO: ILIMITADO`;
            logger("PRIVILEGIOS DE ADMINISTRADOR OTORGADOS", "var(--amber)"); renderAdminPanel(); syncSignalDatabases(); 
        } else {
            if(u.daysLeft <= 0) { alert("TIEMPO DE SUSCRIPCIÓN EXPIRADO. CONTACTA AL ADMINISTRADOR."); currentUser = null; localStorage.removeItem('CORTEX_SESSION'); location.reload(); return; }
            document.getElementById('user-timer').style.display = 'flex'; document.getElementById('timer-days').innerText = u.daysLeft;
            document.getElementById('osd-name').innerText = `BIENVENIDO, ${u.name || 'USUARIO'}`; document.getElementById('ai-count').innerText = `ACCESO: ILIMITADO`; logger(`SESIÓN INICIADA: ${u.email}`, "var(--cyan)");
        }
    }

    function openProfile() { if(!currentUser) return; document.getElementById('profile-modal').style.display = 'flex'; document.getElementById('prof-name').value = currentUser.name || ''; updateCardUI(); }
    function closeProfile() { document.getElementById('profile-modal').style.display = 'none'; }

    let tempPhotoData = '';
    function handlePhotoUpload(event) {
        const file = event.target.files[0]; if(!file) return; const reader = new FileReader();
        reader.onload = function(e) { tempPhotoData = e.target.result; document.getElementById('card-photo').src = tempPhotoData; }; reader.readAsDataURL(file);
    }

    function saveProfile() {
        currentUser.name = document.getElementById('prof-name').value || 'OPERADOR'; if(tempPhotoData) currentUser.photo = tempPhotoData;
        const index = usersDB.findIndex(u => u.email === currentUser.email); if(index !== -1) usersDB[index] = currentUser; saveUsers(); updateCardUI();
        if(currentUser.role !== 'admin') document.getElementById('osd-name').innerText = `BIENVENIDO, ${currentUser.name}`;
    }

    function updateCardUI() {
        document.getElementById('card-name').innerText = (currentUser.name || 'DESCONOCIDO').toUpperCase(); document.getElementById('card-email').innerText = currentUser.email;
        const roleEl = document.getElementById('card-role');
        if(currentUser.role === 'admin') { roleEl.innerText = "OVERLORD / ADMIN"; roleEl.style.background = "var(--amber)"; } 
        else { roleEl.innerText = "OPERADOR ESTÁNDAR"; roleEl.style.background = "var(--cyan)"; roleEl.style.color = "#000"; }
        document.getElementById('card-photo').src = currentUser.photo || "https://via.placeholder.com/80/000000/00f3ff?text=SYS";
    }

    function toggleChat() {
        const chat = document.getElementById('chat-overlay');
        if(chat.style.display === 'none' || chat.style.display === '') { chat.style.display = 'flex'; document.getElementById('chat-title').innerText = `[ COM: ${currentChannelName.substring(0, 15)}... ]`; renderChat(); } 
        else { chat.style.display = 'none'; }
    }

    function renderChat() {
        const container = document.getElementById('chat-messages'); container.innerHTML = '';
        if(!currentChannelName || !chatDB[currentChannelName]) return container.innerHTML = '<div style="color:#666; font-size:0.8rem; text-align:center;">SIN TRANSMISIONES PREVIAS</div>';
        chatDB[currentChannelName].forEach(m => {
            const div = document.createElement('div'); div.className = 'msg-line'; const adminClass = m.role === 'admin' ? 'admin' : '';
            div.innerHTML = `<span class="msg-user ${adminClass}">${m.user}</span><span class="msg-time">[${m.time}]</span>: <span style="color:#fff;">${m.text}</span>`; container.appendChild(div);
        });
        container.scrollTop = container.scrollHeight;
    }

    function sendChatMessage() {
        if(!currentUser || !currentChannelName) return;
        const input = document.getElementById('chat-input'); const text = input.value.trim(); if(!text) return;
        if(!chatDB[currentChannelName]) chatDB[currentChannelName] = [];
        const time = new Date().toLocaleTimeString('en-US', { hour12: false, hour: "numeric", minute: "numeric" });
        const senderName = currentUser.name && currentUser.name !== 'OPERADOR NUEVO' ? currentUser.name : currentUser.email.split('@')[0];
        chatDB[currentChannelName].push({ user: senderName.toUpperCase(), role: currentUser.role, text: text, time: time });
        localStorage.setItem('CORTEX_CHAT', JSON.stringify(chatDB)); input.value = ''; renderChat();
    }
    function handleChatPress(e) { if(e.key === 'Enter') sendChatMessage(); }

    function modifyDays(email, amount) { const u = usersDB.find(x => x.email === email); if(u) { u.daysLeft = (u.daysLeft || 0) + amount; if(u.daysLeft < 0) u.daysLeft = 0; saveUsers(); renderAdminPanel(); } }
    function blockUser(email) { const u = usersDB.find(x => x.email === email); if(u) { u.status = 'blocked'; saveUsers(); logger(`USUARIO BLOQUEADO: ${email}`, "var(--red)"); renderAdminPanel(); } }

    function renderAdminPanel() {
        const panel = document.getElementById('pending-users'); const approvedPanel = document.getElementById('approved-users-list'); const blockedPanel = document.getElementById('blocked-users-list'); 
        const pending = usersDB.filter(u => u.status === 'pending');
        if(pending.length === 0) panel.innerHTML = "<div style='color:var(--green); margin-top:10px;'>NO HAY SOLICITUDES PENDIENTES</div>";
        else {
            let html = "<div style='color:var(--cyan); margin-bottom:5px; margin-top:10px;'>SOLICITUDES:</div>";
            pending.forEach(p => { html += `<div class="user-req"><span>${p.email}</span><button style="background:var(--green); color:#000; border:none; padding:3px 8px; cursor:pointer;" onclick="approveUser('${p.email}')">APROBAR</button></div>`; }); panel.innerHTML = html;
        }
        const approved = usersDB.filter(u => u.status === 'approved' && u.role !== 'admin');
        if(approved.length === 0) approvedPanel.innerHTML = "VACÍO";
        else { 
            approvedPanel.innerHTML = approved.map(u => `
                <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid rgba(0,243,255,0.2); padding:4px 0;">
                    <span style="color:var(--cyan);">> ${u.email} [<span style="color:var(--amber);">${u.daysLeft || 0}d</span>]</span>
                    <div style="display:flex; gap:5px;">
                        <button style="background:var(--green); color:#000; border:none; padding:2px 8px; cursor:pointer; font-weight:bold; border-radius:2px;" onclick="modifyDays('${u.email}', 1)">+1</button>
                        <button style="background:var(--red); color:#000; border:none; padding:2px 8px; cursor:pointer; font-weight:bold; border-radius:2px;" onclick="modifyDays('${u.email}', -1)">-1</button>
                        <button style="background:var(--red); color:#fff; border:none; padding:2px 8px; cursor:pointer; font-weight:bold; border-radius:2px;" onclick="blockUser('${u.email}')">🚫</button>
                    </div>
                </div>
            `).join(''); 
        }
        const blocked = usersDB.filter(u => u.status === 'blocked');
        if(blocked.length === 0) blockedPanel.innerHTML = "VACÍO";
        else {
            blockedPanel.innerHTML = blocked.map(u => `
                <div style="display:flex; justify-content:space-between; align-items:center; border-bottom:1px solid rgba(255,0,85,0.2); padding:4px 0;">
                    <span style="color:var(--red);">> ${u.email}</span>
                    <button style="background:var(--amber); color:#000; border:none; padding:2px 8px; cursor:pointer; font-weight:bold; border-radius:2px;" onclick="approveUser('${u.email}')">RESTAURAR</button>
                </div>
            `).join('');
        }
    }

    function approveUser(email) { const u = usersDB.find(x => x.email === email); if(u) { u.status = 'approved'; saveUsers(); logger(`USUARIO APROBADO/RESTAURADO: ${email}`, 'var(--green)'); renderAdminPanel(); } }

    function runAiSupervisor() {
        if(iaRepairDB.length > 0) { iaRepairDB = []; localStorage.setItem('CORTEX_IA_REPAIR_DB', JSON.stringify(iaRepairDB)); }
        let solidDB = DB.filter(c => c.status !== 'dead'); solidDB = solidDB.map((c, index) => { c.number = index + 1; return c; });
        localStorage.setItem('CORTEX_SOLID_DB', JSON.stringify(solidDB)); alert(`Supervisor IA ejecutado.\nHa analizado los datos de reparación y guardado una BD Estable con ${solidDB.length} nodos.`);
    }

    function viewErrorDB() {
        if(errorDB.length === 0) return alert("No hay errores registrados."); errorDB.slice(-10).forEach(e => { logger(`FALLO: ${e.name} | TIPO: ${e.type} | HORA: ${e.time}`, "var(--red)"); });
    }

    function logger(msg, color = 'var(--green)') {
        const log = document.getElementById('log'); const entry = document.createElement('div'); const time = new Date().toLocaleTimeString('en-US', { hour12: false, hour: "numeric", minute: "numeric", second: "numeric" });
        entry.innerHTML = `<span style="opacity:0.5">[${time}]</span> > <span style="color:${color}">${msg}</span>`; log.appendChild(entry); log.scrollTop = log.scrollHeight; if(log.children.length > 20) log.removeChild(log.firstChild);
    }

    function changeRegion(region) { localStorage.setItem('CORTEX_REGION', region); loadNodes(region); }

    function hashString(str) {
        let hash = 0; for (let i = 0; i < str.length; i++) { hash = (hash << 5) - hash + str.charCodeAt(i); hash |= 0; }
        return Math.abs(hash).toString(16).substring(0, 4).toUpperCase();
    }

    async function quadCoreParseM3U(txt, region, isBackgroundUpdate) {
        logger(`[MOTOR IA] INICIANDO NÚCLEOS VIRTUALES PARA MATRIZ...`, 'var(--cyan)'); const lines = txt.split('\n'); let tempDB = [];
        tempDB.push({ name: "CÁMARA [000]: SEÑAL EXPERIMENTAL AV1/VP9", url: "av1-signal-test", logo: "https://via.placeholder.com/40/00f3ff/000000?text=AV1", status: "live", number: "000", region: region });

        if (region === 'co' || region === 'index') {
            const vipNodes = [
                { name: "Caracol HD", url: "http://199.180.114.174:84/ccaracol.m3u8", number: "102", logo: "https://upload.wikimedia.org/wikipedia/commons/thumb/a/a7/Caracol_HD2.png/512px-Caracol_HD2.png" },
                { name: "RCN HD", url: "http://199.180.114.174:84/crcn.m3u8", number: "104", logo: "https://upload.wikimedia.org/wikipedia/commons/thumb/0/08/Canal_RCN_logo.svg/512px-Canal_RCN_logo.svg.png" },
                { name: "Canal Uno", url: "http://cdns840stu1010.multistream.net/rtvcCanalUNOlive/amlst:live/master.m3u8", number: "105", logo: "https://via.placeholder.com/40/000000/00f3ff?text=UNO" }
            ];
            const formattedVipNodes = vipNodes.map(node => ({ ...node, status: "unk", region: "co" })); tempDB = tempDB.concat(formattedVipNodes);
        }

        const chunkSize = Math.ceil(lines.length / 4);
        for(let motor = 0; motor < 4; motor++) {
            const start = motor * chunkSize; const end = Math.min(start + chunkSize, lines.length);
            let current = {};
            for (let i = start; i < end; i++) {
                const l = lines[i].trim();
                if(l.startsWith('#EXTINF:')) {
                    current.name = l.split(',').length > 1 ? l.split(',').slice(1).join(',').trim() : "NODO DESCONOCIDO";
                    const logoMatch = l.match(/tvg-logo="([^"]+)"/); current.logo = logoMatch ? logoMatch[1] : '';
                    const idMatch = l.match(/tvg-id="([^"]+)"/); const chnoMatch = l.match(/tvg-chno="([^"]+)"/);
                    if (chnoMatch && chnoMatch[1]) { current.number = chnoMatch[1]; } else if (idMatch && idMatch[1]) { current.number = idMatch[1]; } else { current.number = hashString(current.name); }
                    current.status = 'unk'; 
                } else if(l.startsWith('http')) { current.url = l; current.region = region; if (current.name) tempDB.push({...current}); current = {}; }
            }
        }
        
        DB = tempDB; saveMatrixToLocal(region, DB);
        if(!isBackgroundUpdate) { render(true); syncSignalDatabases(); logger(`[MOTOR IA] MATRIZ COMPILADA. 0% LAG.`, 'var(--green)'); } else { render(); }
    }

    async function fetchNetworkMatrix(region, isBackgroundUpdate) {
        let url = "";
        if(region === 'index') url = "https://iptv-org.github.io/iptv/index.m3u";
        else if(region === 'index_nsfw') url = "https://iptv-org.github.io/iptv/index.nsfw.m3u";
        else if(region.startsWith('cat_')) url = `https://iptv-org.github.io/iptv/categories/${region.split('_')[1]}.m3u`;
        else url = `https://iptv-org.github.io/iptv/countries/${region}.m3u`;

        try {
            const res = await fetch(url); if (!res.ok) throw new Error("HTTP Status " + res.status);
            const txt = await res.text(); quadCoreParseM3U(txt, region, isBackgroundUpdate);
        } catch (e) { 
            if(!isBackgroundUpdate) document.getElementById('list').innerHTML = '<div style="text-align:center; padding: 20px; color: var(--red);">ERROR DE CONEXIÓN CON LA COLMENA</div>'; 
            else logger(`FALLO AL ACTUALIZAR BD EN SEGUNDO PLANO.`, 'var(--red)');
        }
    }

    async function loadNodes(region) {
        logger(`ACCEDIENDO A MATRIZ DE DATOS: ${region.toUpperCase()}`, 'var(--amber)');
        document.getElementById('list').innerHTML = '<div style="text-align:center; padding: 20px; color: var(--cyan);">CONECTANDO AL NÚCLEO...</div>';
        const localData = await getMatrixFromLocal(region);
        if (localData && localData.length > 0) { logger(`CACHÉ LOCAL ENCONTRADA. CARGANDO AL INSTANTE.`, 'var(--green)'); DB = localData; render(true); syncSignalDatabases(); fetchNetworkMatrix(region, true); } 
        else { logger(`CACHÉ VACÍA. DESCARGANDO MATRIZ DE RED (ESTO PUEDE TARDAR)...`, 'var(--amber)'); fetchNetworkMatrix(region, false); }
    }

    function render(resetPage = false) {
        if (resetPage) currentPage = 1;
        const list = document.getElementById('list'); const filter = document.getElementById('search').value.toLowerCase();
        let mappedDB = DB.map((c, originalIdx) => ({ ...c, originalIdx }));
        let filteredDB = mappedDB; if (filter) { filteredDB = mappedDB.filter(c => c.name.toLowerCase().includes(filter)); }

        const totalPages = Math.ceil(filteredDB.length / itemsPerPage) || 1; if (currentPage > totalPages) currentPage = totalPages; 
        const startIndex = (currentPage - 1) * itemsPerPage; const endIndex = startIndex + itemsPerPage; const pageItems = filteredDB.slice(startIndex, endIndex);
        const fragment = document.createDocumentFragment();

        pageItems.forEach((c) => {
            const i = c.originalIdx; const card = document.createElement('div'); card.className = `card ${c.status === 'dead' ? 'dead' : ''} ${i === currentPlayingIdx ? 'active' : ''}`; card.style.order = c.status === 'dead' ? 10 : 1;
            const defaultLogo = "https://via.placeholder.com/40/000000/00f3ff?text=CAM"; const displayLogo = c.logo || defaultLogo; const displayNumber = `CAM-${c.number}`; 

            card.innerHTML = `
                <div style="font-family: var(--font-ui); font-size: 0.7rem; color: var(--cyan); margin-right: 10px; opacity: 0.7; flex-shrink: 0; font-weight:bold; min-width: 60px;">[${displayNumber}]</div>
                <img src="${displayLogo}" style="width: 30px; height: 30px; border-radius: 4px; border: 1px solid rgba(0,243,255,0.3); margin-right: 10px; object-fit: contain; background: #000; flex-shrink: 0;" onerror="this.src='${defaultLogo}'">
                <div class="status-dot st-${c.status}"></div>
                <div style="flex:1; overflow: hidden;">
                    <div style="font-size:0.95rem; white-space: nowrap; overflow: hidden; text-overflow: ellipsis;">${c.name}</div>
                    <div style="font-size:0.65rem; color:#888; margin-top: 3px;">ESTADO: <span style="color: ${c.status === 'live' ? 'var(--green)' : c.status === 'dead' ? 'var(--red)' : 'var(--amber)'}">${c.status.toUpperCase()}</span></div>
                </div>`;
            card.onclick = () => play(DB[i], i); fragment.appendChild(card);
        });
        
        list.innerHTML = ''; if (pageItems.length === 0) { list.innerHTML = '<div style="text-align:center; padding: 20px; color: var(--red);">NO SE ENCONTRARON SEÑALES</div>'; } else { list.appendChild(fragment); }
        if(!currentUser || currentUser.role !== 'admin') { document.getElementById('ai-count').innerText = `TOTAL: ${filteredDB.length} NODO(S) | PÁG ${currentPage}/${totalPages}`; }
        renderPagination(totalPages);
    }

    function renderPagination(totalPages) {
        const pagContainer = document.getElementById('pagination-controls'); if (!pagContainer) return;
        pagContainer.innerHTML = `
            <button class="page-btn" onclick="changePage(-1)" ${currentPage === 1 ? 'disabled' : ''}>[ < ANT ]</button>
            <span style="font-size: 0.75rem; color: var(--cyan); font-family: var(--font-ui); font-weight:bold;">PÁGINA ${currentPage} / ${totalPages}</span>
            <button class="page-btn" onclick="changePage(1)" ${currentPage === totalPages ? 'disabled' : ''}>[ SIG > ]</button>
        `;
    }

    function changePage(direction) { currentPage += direction; render(); document.getElementById('list').scrollTop = 0; }

    function updateMediaControls() {
        if(!hls) return; const audioSelect = document.getElementById('audio-tracks'); const subSelect = document.getElementById('subtitle-tracks'); const controlsContainer = document.getElementById('media-controls');
        if(hls.audioTracks && hls.audioTracks.length > 1) { audioSelect.innerHTML = hls.audioTracks.map((t, i) => `<option value="${i}">${t.name || t.lang || 'Audio ' + (i+1)}</option>`).join(''); audioSelect.style.display = 'inline-block'; } else { audioSelect.style.display = 'none'; }
        if(hls.subtitleTracks && hls.subtitleTracks.length > 0) { subSelect.innerHTML = `<option value="-1">Sin Subtítulos</option>` + hls.subtitleTracks.map((t, i) => `<option value="${i}">${t.name || t.lang || 'Sub ' + (i+1)}</option>`).join(''); subSelect.style.display = 'inline-block'; } else { subSelect.style.display = 'none'; }
        controlsContainer.style.display = (audioSelect.style.display === 'inline-block' || subSelect.style.display === 'inline-block') ? 'flex' : 'none';
    }

    function changeAudioTrack(idx) { if(hls) hls.audioTrack = parseInt(idx); } function changeSubtitleTrack(idx) { if(hls) hls.subtitleTrack = parseInt(idx); }

    function play(canal, idx) {
        if(!currentUser) { alert("SISTEMA BLOQUEADO. INICIA SESIÓN."); return; }
        currentPlayingIdx = idx; currentChannelName = canal.name; render();
        if(aiRecoveryTimeout) { clearTimeout(aiRecoveryTimeout); aiRecoveryTimeout = null; }

        setAIAgentState(false); hideAdminTelemetry();
        document.getElementById('bb-channel-name').innerText = `[CAM-${canal.number}] ${canal.name}`; document.getElementById('bb-program-info').innerText = `INFO: Conectando a transmisión...`; document.getElementById('ia-sync-status').style.display = 'none';
        document.getElementById('osd-name').innerText = canal.name; document.getElementById('osd-name').style.color = "var(--cyan)"; document.getElementById('osd-msg').innerText = "SINTONIZANDO CÁMARA..."; osdRes.style.display = 'none'; 
        
        document.getElementById('user-action-btns').style.display = 'flex'; document.getElementById('media-controls').style.display = 'none'; 
        if(canal.url === "av1-signal-test") { document.getElementById('btn-av1-info').style.display = 'inline-block'; } else { document.getElementById('btn-av1-info').style.display = 'none'; }

        showAdminTelemetry(canal.name, false);

        if(document.getElementById('chat-overlay').style.display !== 'none') { document.getElementById('chat-title').innerText = `[ COM: ${currentChannelName.substring(0, 15)}... ]`; renderChat(); }

        const mobFloating = document.getElementById('mobile-floating-node'); const mobFloatingName = document.getElementById('mobile-floating-name');
        if (mobFloating && mobFloatingName) { mobFloating.classList.add('active'); mobFloatingName.innerText = `[CAM-${canal.number}] ${canal.name}`; }

        logger(`[IA CORE] AUMENTANDO POTENCIA DE SEÑAL...`, "var(--cyan)");

        if(hls) { hls.stopLoad(); hls.detachMedia(); hls.destroy(); hls = null; }
        videoCore.pause(); videoAV1.pause();
        
        if(canal.url === "av1-signal-test") {
            videoCore.style.display = 'none'; videoAV1.style.display = 'block'; playerBox.classList.add('signal-glow');
            videoAV1.load(); videoAV1.play().catch(() => {}); updateStatus(idx, 'live');
            document.getElementById('osd-msg').innerText = "TRANSMISIÓN ESTABLE // PANEL AV1"; resetInactivity(); return;
        }

        videoCore.style.display = 'block'; videoAV1.style.display = 'none'; playerBox.classList.remove('signal-glow'); videoCore.innerHTML = ''; videoCore.removeAttribute('src');

        if (Hls.isSupported()) {
            hls = new Hls({ enableWorker: true, maxBufferLength: 60, maxMaxBufferLength: 1200, maxBufferSize: 120 * 1000 * 1000, liveSyncDurationCount: 5, fragLoadingMaxRetry: 10, fragLoadingRetryDelay: 500, levelLoadingMaxRetry: 6 });
            hls.loadSource(canal.url); hls.attachMedia(videoCore);
            hls.on(Hls.Events.MANIFEST_PARSED, () => {
                if(aiRecoveryTimeout) { clearTimeout(aiRecoveryTimeout); aiRecoveryTimeout = null; } 
                videoCore.play().catch(e => logger("AUTOPLAY BLOQUEADO", 'var(--amber)')); updateStatus(idx, 'live'); document.getElementById('osd-msg').innerText = "TRANSMISIÓN ESTABLE // HLS";
                setAIAgentState(false); updateMediaControls(); 
            });
            hls.on(Hls.Events.AUDIO_TRACKS_UPDATED, updateMediaControls); hls.on(Hls.Events.SUBTITLE_TRACKS_UPDATED, updateMediaControls);
            hls.on(Hls.Events.ERROR, (event, data) => {
                if (data.fatal) {
                    document.getElementById('bb-program-info').innerText = `INFO: Error en transmisión. Intentando recuperar...`;
                    if(!aiRecoveryTimeout) { aiRecoveryTimeout = setTimeout(() => { logger(`[IA AUTO-SKIP] NODO INESTABLE. SALTANDO A CÁMARA SIGUIENTE...`, "var(--amber)"); updateStatus(idx, 'dead'); aiRecoveryTimeout = null; playNextChannel(); }, 2000); }
                    let specificReason = "FALLO DESCONOCIDO EN EL ORIGEN.";
                    if (data.type === Hls.ErrorTypes.NETWORK_ERROR) { 
                        if(data.details === Hls.ErrorDetails.MANIFEST_LOAD_TIMEOUT) specificReason = "TIMEOUT: El servidor no responde o está saturado."; else if(data.details === Hls.ErrorDetails.MANIFEST_LOAD_ERROR) specificReason = "LOAD ERROR: El enlace M3U8 está caído o bloqueado (CORS)."; else specificReason = "NETWORK ERROR: Pérdida de conexión con el nodo.";
                        setAIAgentState(true, "ERROR DE RED // RECUPERANDO..."); showAdminTelemetry(canal.name, true, specificReason); logChannelErrorAndFix(canal.name, canal.url, "NETWORK_ERROR", specificReason); hls.startLoad(); 
                    } else if (data.type === Hls.ErrorTypes.MEDIA_ERROR) { 
                        if(data.details === Hls.ErrorDetails.BUFFER_STALLED_ERROR) specificReason = "BUFFER STALLED: Falta de ancho de banda o fragmentos rotos."; else specificReason = "MEDIA ERROR: Formato de video irreconocible o corrupto.";
                        setAIAgentState(true, "FALLO DE CODEC // RECOMPILANDO..."); showAdminTelemetry(canal.name, true, specificReason); logChannelErrorAndFix(canal.name, canal.url, "MEDIA_ERROR", specificReason); hls.recoverMediaError(); 
                    } else { 
                        specificReason = "FATAL ERROR: El canal ha dejado de emitir permanentemente."; setAIAgentState(true, "NODO MUERTO // ARCHIVADO"); showAdminTelemetry(canal.name, true, specificReason); logChannelErrorAndFix(canal.name, canal.url, "FATAL_ERROR", specificReason); updateStatus(idx, 'dead'); hls.destroy(); 
                    }
                }
            });
        } else if (videoCore.canPlayType('application/vnd.apple.mpegurl')) {
            videoCore.src = canal.url;
            videoCore.addEventListener('loadedmetadata', () => { if(aiRecoveryTimeout) { clearTimeout(aiRecoveryTimeout); aiRecoveryTimeout = null; } videoCore.play(); updateStatus(idx, 'live'); setAIAgentState(false); });
            videoCore.addEventListener('error', () => {
                setAIAgentState(true, "ERROR NATIVO // ARCHIVADO"); showAdminTelemetry(canal.name, true, "NATIVE ERROR: El reproductor de Apple Safari no puede decodificar la fuente."); logChannelErrorAndFix(canal.name, canal.url, "NATIVE_ERROR", "Fallo nativo Safari/iOS"); updateStatus(idx, 'dead'); document.getElementById('bb-program-info').innerText = `INFO: Señal muerta (Archivado).`;
                if(!aiRecoveryTimeout) { aiRecoveryTimeout = setTimeout(() => { playNextChannel(); }, 2000); }
            });
        }
        resetInactivity();
    }
    
    function updateStatus(i, s) { 
        if(DB[i] && DB[i].status !== s) { 
            DB[i].status = s; 
            if (s === 'live' && currentUser) {
                document.getElementById('bb-program-info').innerText = `INFO: Transmisión Estable // Verificado y Respaldado por IA`; document.getElementById('ia-sync-status').style.display = 'flex';
                if (!currentUser.workingNodes) currentUser.workingNodes = [];
                if (!currentUser.workingNodes.find(c => c.url === DB[i].url)) {
                    let canalToSave = {...DB[i], region: document.getElementById('region').value}; currentUser.workingNodes.push(canalToSave); saveUsers(); if(currentSubPanel === 'auto') openSubPanel('auto'); 
                }
            }
            render(); syncSignalDatabases(); 
        } 
    }
</script>
</body>
</html>
