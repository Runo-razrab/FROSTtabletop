'''jsx
import React, { useState, useEffect, useRef } from 'react';

const App = () => {
  // State management
  const [activeTab, setActiveTab] = useState('maps');
  const [showDMTools, setShowDMTools] = useState(false);
  const [diceRolls, setDiceRolls] = useState([]);
  const [chatMessages, setChatMessages] = useState([]);
  const [selectedCharacter, setSelectedCharacter] = useState(null);
  const [activeMap, setActiveMap] = useState(1);
  const [showSpellEffects, setShowSpellEffects] = useState(false);
  const [show3DMiniatures, setShow3DMiniatures] = useState(true);
  const [selectedTemplate, setSelectedTemplate] = useState('basic');
  const [selectedTokenId, setSelectedTokenId] = useState(null);
  const [availableWidth, setAvailableWidth] = useState(0);
  const [availableHeight, setAvailableHeight] = useState(0);
  const [customMapImage, setCustomMapImage] = useState(null);
  const [isUploading, setIsUploading] = useState(false);
  
  // Refs
  const mapContainerRef = useRef(null);
  
  // Mock data
  const characters = [
    { id: 1, name: 'Gandalf', class: 'Wizard', level: 10, imageUrl: 'https://picsum.photos/200/300?random=1' },
    { id: 2, name: 'Aragorn', class: 'Fighter', level: 8, imageUrl: 'https://picsum.photos/200/300?random=2' },
    { id: 3, name: 'Legolas', class: 'Ranger', level: 9, imageUrl: 'https://picsum.photos/200/300?random=3' },
  ];
  
  const maps = [
    { id: 1, name: 'Misty Mountains', layers: ['surface', 'underground'] },
    { id: 2, name: 'Mirkwood Forest' },
    { id: 3, name: 'Rivendell' },
  ];
  
  const spellTemplates = [
    { id: 'basic', name: 'Basic Character' },
    { id: 'wizard', name: 'Wizard Template' },
    { id: 'fighter', name: 'Fighter Template' },
  ];
  
  const monsters = [
    { id: 1, name: 'Orc', ac: 13, hp: 15, cr: 1/2 },
    { id: 2, name: 'Skeleton', ac: 13, hp: 13, cr: 1/4 },
    { id: 3, name: 'Goblin', ac: 15, hp: 7, cr: 1/4 },
  ];
  
  // Token positions state
  const [tokens, setTokens] = useState([
    { id: 1, name: 'G', x: 10, y: 10, color: 'bg-red-500' },
    { id: 2, name: 'A', x: 32, y: 20, color: 'bg-blue-500' },
    { id: 3, name: 'L', x: 20, y: 40, color: 'bg-green-500' },
  ]);
  
  // Function to handle dice rolling
  const rollDice = (sides) => {
    const result = Math.floor(Math.random() * sides) + 1;
    setDiceRolls([...diceRolls, { sides, result, timestamp: new Date() }]);
  };
  
  // Function to send chat messages
  const sendMessage = (message) => {
    if (message.trim()) {
      setChatMessages([...chatMessages, { 
        user: 'Player', 
        text: message, 
        timestamp: new Date() 
      }]);
    }
  };
  
  // Custom hook for keyboard shortcuts
  useEffect(() => {
    const handleKeyDown = (e) => {
      if (e.ctrlKey && e.key === 'm') {
        setShowDMTools(!showDMTools);
      }
    };
    
    window.addEventListener('keydown', handleKeyDown);
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, [showDMTools]);
  
  // Keyboard movement handler for tokens
  useEffect(() => {
    const handleKeyDown = (e) => {
      if (!selectedTokenId) return;
      
      const step = 10;
      setTokens(prevTokens => prevTokens.map(token => {
        if (token.id === selectedTokenId) {
          let newX = token.x;
          let newY = token.y;
          
          switch (e.key.toLowerCase()) {
            case 'w': // Move up
              newY = Math.max(0, token.y - step);
              break;
            case 's': // Move down
              newY = Math.min(availableHeight - 40, token.y + step);
              break;
            case 'a': // Move left
              newX = Math.max(0, token.x - step);
              break;
            case 'd': // Move right
              newX = Math.min(availableWidth - 40, token.x + step);
              break;
            default:
              return token;
          }
          
          return { ...token, x: newX, y: newY };
        }
        return token;
      }));
    };
    
    window.addEventListener('keydown', handleKeyDown);
    return () => {
      window.removeEventListener('keydown', handleKeyDown);
    };
  }, [selectedTokenId, availableWidth, availableHeight]);
  
  // Measure map container size
  useEffect(() => {
    const updateSize = () => {
      if (mapContainerRef.current) {
        const rect = mapContainerRef.current.getBoundingClientRect();
        setAvailableWidth(rect.width);
        setAvailableHeight(rect.height);
      }
    };
    
    window.addEventListener('resize', updateSize);
    updateSize();
    
    return () => window.removeEventListener('resize', updateSize);
  }, []);
  
  // Add new token function
  const addToken = () => {
    const newId = Math.max(...tokens.map(t => t.id)) + 1;
    setTokens([...tokens, {
      id: newId,
      name: String.fromCharCode(64 + newId),
      x: 50,
      y: 50,
      color: ['bg-red-500', 'bg-blue-500', 'bg-green-500', 'bg-yellow-500'][newId % 4]
    }]);
  };
  
  // Handle map image upload
  const handleMapImageUpload = (e) => {
    const file = e.target.files[0];
    if (file) {
      setIsUploading(true);
      const reader = new FileReader();
      reader.onload = (e) => {
        setCustomMapImage(e.target.result);
        setIsUploading(false);
        // Reset token positions when new image is uploaded
        setTokens(tokens.map(token => ({ ...token, x: 50, y: 50 })));
      };
      reader.readAsDataURL(file);
    }
  };
  
  // Remove custom map image
  const removeMapImage = () => {
    setCustomMapImage(null);
    // Reset token positions
    setTokens(tokens.map(token => ({ ...token, x: 10, y: 10 })));
  };
  
  // SVG Icons
  const DiceIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path d="M9 3.5a1 1 0 112 0 1 1 0 01-2 0zM6.354 5.5a1 1 0 100 2 1 1 0 000-2zM13.646 5.5a1 1 0 100 2 1 1 0 000-2z" />
      <path fillRule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-8-6a6 6 0 00-6 6c0 1.137.36 2.185.958 3.042l-.003-.003.003-.003A5.994 5.994 0 0010 4zM4 10a6 6 0 016-6v.055A5.96 5.96 0 004 10H4z" clipRule="evenodd" />
    </svg>
  );
  
  const ChatIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path fillRule="evenodd" d="M18 5v8a2 2 0 01-2 2h-5l-5 4v-4H4a2 2 0 01-2-2V5a2 2 0 012-2h12a2 2 0 012 2zM7 8a1 1 0 012 0v.01a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v.01a1 1 0 102 0V8a1 1 0 00-1-1z" clipRule="evenodd" />
    </svg>
  );
  
  const MapIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path fillRule="evenodd" d="M6 2a2 2 0 00-2 2v12a2 2 0 002 2h8a2 2 0 002-2V4a2 2 0 00-2-2H6zm1 2a1 1 0 000 2h6a1 1 0 100-2H7zm6 7a1 1 0 00-1 1v3a1 1 0 102 0v-3a1 1 0 00-1-1z" clipRule="evenodd" />
    </svg>
  );
  
  const CharacterIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path fillRule="evenodd" d="M10 9a3 3 0 100-6 3 3 0 000 6zm-7 9a7 7 0 1114 0H3z" clipRule="evenodd" />
    </svg>
  );
  
  const DMIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path fillRule="evenodd" d="M18 10a8 8 0 11-16 0 8 8 0 0116 0zm-8-3a1 1 0 00-.867.5 1 1 0 11-1.731-1A3 3 0 0113 8a3.001 3.001 0 01-2 2.83V11a1 1 0 11-2 0v-1a1 1 0 011-1 1 1 0 100-2zm0 8a1 1 0 100-2 1 1 0 000 2z" clipRule="evenodd" />
    </svg>
  );
  
  const SpellEffectIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path fillRule="evenodd" d="M4 3a2 2 0 00-2 2v10a2 2 0 002 2h12a2 2 0 002-2V5a2 2 0 00-2-2H4zm10 10a1 1 0 11-2 0 1 1 0 012 0z" clipRule="evenodd" />
    </svg>
  );
  
  const ThreeDIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path d="M10 3.5a1.5 1.5 0 013 0 1.5 1.5 0 01-3 0zM3.5 10a1.5 1.5 0 011.5 1.5 1.5 1.5 0 01-1.5-1.5 1.5 1.5 0 011.5-1.5 1.5 1.5 0 010 3zM18 10a1.5 1.5 0 01-1.5 1.5 1.5 1.5 0 010-3 1.5 1.5 0 011.5 1.5z" />
    </svg>
  );
  
  const UploadIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path fillRule="evenodd" d="M3 17a1 1 0 011-1h12a1 1 0 110 2H4a1 1 0 01-1-1zm3.293-7.707a1 1 0 011.414-1.414l3 3a1 1 0 010 1.414l-3 3a1 1 0 01-1.414-1.414L7.586 10 5.293 7.707zm5 0a1 1 0 011.414-1.414l3 3a1 1 0 010 1.414l-3 3a1 1 0 11-1.414-1.414L12.586 10l-2.293-2.293z" clipRule="evenodd" />
    </svg>
  );
  
  const TrashIcon = () => (
    <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5" viewBox="0 0 20 20" fill="currentColor">
      <path fillRule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clipRule="evenodd" />
    </svg>
  );
  
  // Render the application
  return (
    <div className="min-h-screen bg-gray-900 text-white flex flex-col">
      {/* Header */}
      <header className="bg-gray-800 p-4 shadow-lg">
        <div className="container mx-auto flex justify-between items-center">
          <div className="flex items-center space-x-2">
            <div className="w-8 h-8 bg-indigo-600 rounded-full flex items-center justify-center font-bold">
              DM
            </div>
            <h1 className="text-2xl font-bold text-indigo-400">DungeonMaster</h1>
          </div>
          
          <nav className="hidden md:flex items-center space-x-1">
            <button 
              onClick={() => setActiveTab('maps')} 
              className={`flex items-center px-4 py-2 rounded-md transition-colors ${
                activeTab === 'maps' 
                  ? 'bg-indigo-600 text-white' 
                  : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
              }`}
            >
              <MapIcon />
              <span className="ml-2">Maps</span>
            </button>
            <button 
              onClick={() => setActiveTab('characters')} 
              className={`flex items-center px-4 py-2 rounded-md transition-colors ${
                activeTab === 'characters' 
                  ? 'bg-indigo-600 text-white' 
                  : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
              }`}
            >
              <CharacterIcon />
              <span className="ml-2">Characters</span>
            </button>
            <button 
              onClick={() => setActiveTab('chat')} 
              className={`flex items-center px-4 py-2 rounded-md transition-colors ${
                activeTab === 'chat' 
                  ? 'bg-indigo-600 text-white' 
                  : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
              }`}
            >
              <ChatIcon />
              <span className="ml-2">Chat</span>
            </button>
            <button 
              onClick={() => setShowDMTools(!showDMTools)} 
              className={`flex items-center px-4 py-2 rounded-md transition-colors ${
                showDMTools 
                  ? 'bg-indigo-600 text-white' 
                  : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
              }`}
              title="Toggle DM Tools (Ctrl+M)"
            >
              <DMIcon />
              <span className="ml-2">DM Tools</span>
            </button>
          </nav>
          
          <div className="flex items-center space-x-2">
            <button 
              onClick={() => setShowSpellEffects(!showSpellEffects)} 
              className={`p-2 rounded-md ${
                showSpellEffects 
                  ? 'bg-indigo-600 text-white' 
                  : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
              }`}
              title="Toggle Spell Effects"
            >
              <SpellEffectIcon />
            </button>
            <button 
              onClick={() => setShow3DMiniatures(!show3DMiniatures)} 
              className={`p-2 rounded-md ${
                show3DMiniatures 
                  ? 'bg-indigo-600 text-white' 
                  : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
              }`}
              title="Toggle 3D Miniatures"
            >
              <ThreeDIcon />
            </button>
          </div>
        </div>
      </header>
      
      {/* Mobile Navigation */}
      <div className="md:hidden bg-gray-800 p-2">
        <div className="flex justify-around">
          <button 
            onClick={() => setActiveTab('maps')} 
            className={`flex flex-col items-center px-3 py-2 rounded-md transition-colors ${
              activeTab === 'maps' 
                ? 'bg-indigo-600 text-white' 
                : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
            }`}
          >
            <MapIcon />
            <span className="text-xs mt-1">Maps</span>
          </button>
          <button 
            onClick={() => setActiveTab('characters')} 
            className={`flex flex-col items-center px-3 py-2 rounded-md transition-colors ${
              activeTab === 'characters' 
                ? 'bg-indigo-600 text-white' 
                : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
            }`}
          >
            <CharacterIcon />
            <span className="text-xs mt-1">Chars</span>
          </button>
          <button 
            onClick={() => setActiveTab('chat')} 
            className={`flex flex-col items-center px-3 py-2 rounded-md transition-colors ${
              activeTab === 'chat' 
                ? 'bg-indigo-600 text-white' 
                : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
            }`}
          >
            <ChatIcon />
            <span className="text-xs mt-1">Chat</span>
          </button>
          <button 
            onClick={() => setShowDMTools(!showDMTools)} 
            className={`flex flex-col items-center px-3 py-2 rounded-md transition-colors ${
              showDMTools 
                ? 'bg-indigo-600 text-white' 
                : 'bg-gray-700 hover:bg-gray-600 text-gray-300'
            }`}
          >
            <DMIcon />
            <span className="text-xs mt-1">DM</span>
          </button>
        </div>
      </div>
      
      {/* Main Content */}
      <main className="flex-grow container mx-auto p-4 flex flex-col">
        {/* Game Board Section */}
        <div className="flex-grow flex flex-col md:flex-row gap-4 mb-4">
          {/* Left Sidebar - Game Tools */}
          <aside className="w-full md:w-1/4 bg-gray-800 rounded-lg p-4 shadow-lg">
            <h2 className="text-xl font-semibold mb-4">Game Tools</h2>
            
            {/* Dice Roller */}
            <div className="mb-6">
              <h3 className="text-lg font-medium mb-2 flex items-center">
                <DiceIcon />
                <span className="ml-2">Dice Roller</span>
              </h3>
              <div className="flex flex-wrap gap-2">
                {[4, 6, 8, 10, 12, 20].map(sides => (
                  <button 
                    key={sides}
                    onClick={() => rollDice(sides)}
                    className="px-3 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors transform hover:scale-105"
                  >
                    d{sides}
                  </button>
                ))}
              </div>
              <div className="mt-4">
                <h4 className="font-medium mb-2">Recent Rolls:</h4>
                {diceRolls.length > 0 ? (
                  <div className="space-y-1">
                    {diceRolls.slice(-5).map((roll, index) => (
                      <div key={index} className="text-sm bg-gray-700 p-2 rounded">
                        d{roll.sides}: <span className="font-medium">{roll.result}</span>
                      </div>
                    ))}
                  </div>
                ) : (
                  <p className="text-sm text-gray-400">No rolls yet</p>
                )}
              </div>
            </div>
            
            {/* Character List */}
            <div>
              <h3 className="text-lg font-medium mb-2 flex items-center">
                <CharacterIcon />
                <span className="ml-2">Characters</span>
              </h3>
              <ul className="space-y-2">
                {characters.map(char => (
                  <li 
                    key={char.id} 
                    onClick={() => setSelectedCharacter(char)}
                    className={`p-2 rounded cursor-pointer transition-colors flex items-center ${
                      selectedCharacter?.id === char.id 
                        ? 'bg-indigo-600' 
                        : 'bg-gray-700 hover:bg-gray-600'
                    }`}
                  >
                    <img 
                      src={char.imageUrl} 
                      alt={char.name} 
                      className="w-8 h-8 rounded-full mr-2 object-cover"
                    />
                    <span>
                      {char.name} - {char.class} {char.level}
                    </span>
                  </li>
                ))}
              </ul>
            </div>
          </aside>
          
          {/* Center Content - Based on active tab */}
          <section className="flex-grow bg-gray-800 rounded-lg p-4 shadow-lg overflow-hidden">
            {activeTab === 'maps' && (
              <div className="h-full flex flex-col">
                <h2 className="text-xl font-semibold mb-4">Maps</h2>
                
                {/* Map Tabs */}
                <div className="flex mb-4 border-b border-gray-700">
                  {maps.map(map => (
                    <button 
                      key={map.id}
                      className={`px-4 py-2 font-medium transition-colors ${
                        activeMap === map.id 
                          ? 'text-indigo-400 border-b-2 border-indigo-400' 
                          : 'text-gray-400 hover:text-indigo-300'
                      }`}
                      onClick={() => setActiveMap(map.id)}
                    >
                      {map.name}
                    </button>
                  ))}
                </div>
                
                {/* Map Display */}
                <div 
                  ref={mapContainerRef}
                  className="relative flex-grow bg-gray-700 rounded overflow-hidden mb-4"
                >
                  {customMapImage ? (
                    <img 
                      src={customMapImage} 
                      alt="Custom Map" 
                      className="absolute inset-0 w-full h-full object-cover transition-opacity duration-300"
                      onLoad={() => setIsUploading(false)}
                    />
                  ) : (
                    <div className="absolute inset-0 flex items-center justify-center">
                      <p className="text-gray-400">Map Display Area</p>
                    </div>
                  )}
                  
                  {/* Tokens (characters, monsters, etc.) */}
                  {tokens.map(token => (
                    <div
                      key={token.id}
                      className={`absolute w-10 h-10 ${token.color} rounded-full flex items-center justify-center text-white font-bold transform transition-transform hover:scale-110 cursor-move ${
                        selectedTokenId === token.id ? 'ring-2 ring-indigo-400' : ''
                      }`}
                      style={{ top: `${token.y}px`, left: `${token.x}px` }}
                      onClick={() => setSelectedTokenId(token.id)}
                    >
                      {token.name}
                    </div>
                  ))}
                  
                  {/* Line of Sight Visualization */}
                  <div className="absolute inset-0 pointer-events-none">
                    <div className="absolute top-0 left-0 w-1/2 h-full bg-black bg-opacity-40"></div>
                    <div className="absolute top-0 right-0 w-1/2 h-1/2 bg-black bg-opacity-40"></div>
                  </div>
                  
                  {/* Uploading overlay */}
                  {isUploading && (
                    <div className="absolute inset-0 flex items-center justify-center bg-black bg-opacity-50">
                      <div className="animate-spin rounded-full h-12 w-12 border-t-2 border-b-2 border-indigo-400"></div>
                    </div>
                  )}
                </div>
                
                {/* Map Controls */}
                <div className="flex flex-wrap gap-2">
                  <button 
                    onClick={addToken}
                    className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors"
                  >
                    Add Token
                  </button>
                  <button className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                    Clear Tokens
                  </button>
                  <button className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                    Toggle Grid
                  </button>
                  <button className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                    Add Monster
                  </button>
                  
                  {/* Upload Map Image Button */}
                  <div className="relative">
                    <input 
                      type="file" 
                      accept="image/*" 
                      onChange={handleMapImageUpload} 
                      className="hidden" 
                      id="mapImageUpload"
                    />
                    <label 
                      htmlFor="mapImageUpload" 
                      className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors cursor-pointer flex items-center"
                    >
                      <UploadIcon />
                      <span className="ml-2">Upload Map Image</span>
                    </label>
                  </div>
                  
                  {/* Remove Map Image Button */}
                  {customMapImage && (
                    <button 
                      onClick={removeMapImage}
                      className="px-4 py-2 bg-red-600 hover:bg-red-700 rounded transition-colors flex items-center"
                    >
                      <TrashIcon />
                      <span className="ml-2">Remove Image</span>
                    </button>
                  )}
                </div>
                
                {/* Movement Instructions */}
                <div className="mt-4 text-sm text-gray-400">
                  <p>Selected token movement: Use <span className="text-indigo-400">W</span> (up), 
                  <span className="text-indigo-400"> S</span> (down), 
                  <span className="text-indigo-400"> A</span> (left), 
                  <span className="text-indigo-400"> D</span> (right)</p>
                </div>
              </div>
            )}
            
            {activeTab === 'characters' && (
              <div className="h-full flex flex-col">
                <h2 className="text-xl font-semibold mb-4">Character Management</h2>
                
                {/* Character Templates */}
                <div className="mb-6">
                  <h3 className="text-lg font-medium mb-2">Character Templates</h3>
                  <div className="flex flex-wrap gap-2">
                    {spellTemplates.map(template => (
                      <button 
                        key={template.id}
                        onClick={() => setSelectedTemplate(template.id)}
                        className={`px-3 py-1 rounded transition-colors ${
                          selectedTemplate === template.id
                            ? 'bg-indigo-600'
                            : 'bg-gray-700 hover:bg-gray-600'
                        }`}
                      >
                        {template.name}
                      </button>
                    ))}
                  </div>
                </div>
                
                {selectedCharacter ? (
                  <div className="grid grid-cols-1 md:grid-cols-3 gap-4 h-full">
                    {/* Character Info */}
                    <div className="md:col-span-1 bg-gray-700 p-4 rounded">
                      <img 
                        src={selectedCharacter.imageUrl} 
                        alt={selectedCharacter.name} 
                        className="w-full h-40 object-cover rounded mb-4"
                      />
                      <h3 className="text-lg font-medium mb-1">{selectedCharacter.name}</h3>
                      <p className="text-sm text-gray-300 mb-4">
                        {selectedCharacter.class} Level {selectedCharacter.level}
                      </p>
                      
                      <div className="space-y-2">
                        <div className="bg-gray-800 p-2 rounded">
                          <p className="text-xs text-gray-400">AC</p>
                          <p className="text-lg">15</p>
                        </div>
                        <div className="bg-gray-800 p-2 rounded">
                          <p className="text-xs text-gray-400">HP</p>
                          <p className="text-lg">85</p>
                        </div>
                        <div className="bg-gray-800 p-2 rounded">
                          <p className="text-xs text-gray-400">Initiative</p>
                          <p className="text-lg">+5</p>
                        </div>
                      </div>
                    </div>
                    
                    {/* Character Stats */}
                    <div className="md:col-span-2 space-y-6">
                      <div>
                        <h4 className="font-medium mb-2">Stats</h4>
                        <div className="grid grid-cols-2 md:grid-cols-3 gap-4">
                          {['Strength', 'Dexterity', 'Constitution', 'Intelligence', 'Wisdom', 'Charisma'].map(stat => (
                            <div key={stat} className="bg-gray-700 p-3 rounded">
                              <span className="text-sm text-gray-400">{stat}</span>
                              <div className="flex items-center mt-1">
                                <span className="font-medium text-lg">{Math.floor(Math.random() * 18) + 1}</span>
                                <span className="ml-2 text-sm text-gray-400">
                                  (+{Math.floor((Math.random() * 4) + 1)})
                                </span>
                              </div>
                            </div>
                          ))}
                        </div>
                      </div>
                      
                      {/* Spells */}
                      <div>
                        <h4 className="font-medium mb-2">Spells</h4>
                        <div className="grid grid-cols-1 md:grid-cols-2 gap-2">
                          {['Fireball', 'Magic Missile', 'Invisibility', 'Fly', 'Counterspell', 'Web', 'Lightning Bolt', 'Teleport'].map(spell => (
                            <div key={spell} className="bg-gray-700 p-3 rounded flex justify-between items-center">
                              <span>{spell}</span>
                              <button className="text-xs px-2 py-1 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                                Cast
                              </button>
                            </div>
                          ))}
                        </div>
                      </div>
                    </div>
                  </div>
                ) : (
                  <div className="flex-grow flex items-center justify-center">
                    <p className="text-gray-400 text-center">
                      Select a character from the list to view details.
                    </p>
                  </div>
                )}
              </div>
            )}
            
            {activeTab === 'chat' && (
              <div className="h-full flex flex-col">
                <h2 className="text-xl font-semibold mb-4">Chat</h2>
                
                {/* Chat Messages */}
                <div className="flex-grow bg-gray-700 rounded p-3 overflow-y-auto mb-4">
                  {chatMessages.length > 0 ? (
                    chatMessages.map((msg, index) => (
                      <div key={index} className="mb-3">
                        <div className="flex items-center mb-1">
                          <span className="w-8 h-8 rounded-full bg-indigo-600 flex items-center justify-center text-xs mr-2">
                            {msg.user.charAt(0)}
                          </span>
                          <span className="text-indigo-400 font-medium">{msg.user}</span>
                          <span className="text-xs text-gray-400 ml-2">
                            {new Date(msg.timestamp).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'})}
                          </span>
                        </div>
                        <div className="ml-10 p-2 bg-gray-800 rounded">
                          {msg.text}
                        </div>
                      </div>
                    ))
                  ) : (
                    <div className="flex items-center justify-center h-full">
                      <p className="text-gray-400">No messages yet</p>
                    </div>
                  )}
                </div>
                
                {/* Chat Input */}
                <div className="flex">
                  <input
                    type="text"
                    placeholder="Type your message..."
                    className="flex-grow p-2 bg-gray-800 border border-gray-600 rounded-l focus:outline-none focus:ring-2 focus:ring-indigo-500"
                    onKeyPress={(e) => {
                      if (e.key === 'Enter') {
                        sendMessage(e.target.value);
                        e.target.value = '';
                      }
                    }}
                  />
                  <button 
                    onClick={() => {
                      const input = document.querySelector('input[type="text"]');
                      if (input.value.trim()) {
                        sendMessage(input.value);
                        input.value = '';
                      }
                    }}
                    className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded-r transition-colors"
                  >
                    Send
                  </button>
                </div>
                
                <div className="mt-4">
                  <h3 className="text-lg font-medium mb-2">Voice Chat</h3>
                  <div className="bg-gray-700 p-4 rounded text-center">
                    <p className="text-gray-400 mb-2">Voice chat integration would be here</p>
                    <button className="px-4 py-2 bg-green-600 hover:bg-green-700 rounded transition-colors flex items-center justify-center mx-auto">
                      <svg xmlns="http://www.w3.org/2000/svg" className="h-5 w-5 mr-2" viewBox="0 0 20 20" fill="currentColor">
                        <path fillRule="evenodd" d="M7 4a3 3 0 016 0v4a3 3 0 11-6 0V4zm4 10.93A7.001 7.001 0 0017 8a1 1 0 10-2 0A5 5 0 015 8a1 1 0 00-2 0 7.001 7.001 0 006 6.93V17H6a1 1 0 100 2h8a1 1 0 100-2h-3v-2.07z" clipRule="evenodd" />
                      </svg>
                      Join Voice
                    </button>
                  </div>
                </div>
              </div>
            )}
          </section>
          
          {/* Right Sidebar - DM Tools (conditionally rendered) */}
          {showDMTools && (
            <aside className="w-full md:w-1/4 bg-gray-800 rounded-lg p-4 shadow-lg">
              <h2 className="text-xl font-semibold mb-4">DM Tools</h2>
              
              <div className="mb-6">
                <h3 className="text-lg font-medium mb-2">Secret Rolls</h3>
                <div className="space-y-2">
                  {diceRolls.slice(0, 3).map((roll, index) => (
                    <div key={index} className="bg-gray-700 p-2 rounded">
                      <p className="text-sm">d{roll.sides}: <span className="font-medium">••••</span></p>
                    </div>
                  ))}
                </div>
                <button 
                  onClick={() => rollDice(20)}
                  className="mt-2 w-full px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors"
                >
                  Secret Roll
                </button>
              </div>
              
              <div className="mb-6">
                <h3 className="text-lg font-medium mb-2">Monsters</h3>
                <ul className="space-y-2">
                  {monsters.map(monster => (
                    <li 
                      key={monster.id} 
                      className="p-2 bg-gray-700 rounded cursor-pointer hover:bg-gray-600 transition-colors"
                    >
                      <div className="font-medium">{monster.name}</div>
                      <div className="text-sm text-gray-300">
                        AC: {monster.ac} • HP: {monster.hp} • CR: {monster.cr}
                      </div>
                    </li>
                  ))}
                </ul>
                <button className="mt-2 w-full px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                  Add Monster
                </button>
              </div>
              
              <div>
                <h3 className="text-lg font-medium mb-2">Notes</h3>
                <textarea
                  placeholder="DM Notes..."
                  className="w-full h-32 p-2 bg-gray-700 border border-gray-600 rounded focus:outline-none focus:ring-2 focus:ring-indigo-500 resize-none"
                ></textarea>
              </div>
            </aside>
          )}
        </div>
        
        {/* Community & Content Sharing Section */}
        <div className="bg-gray-800 rounded-lg p-4 shadow-lg mb-4">
          <h2 className="text-xl font-semibold mb-4">Community & Content Sharing</h2>
          
          <div className="grid grid-cols-1 md:grid-cols-3 gap-4">
            <div className="bg-gray-700 p-4 rounded">
              <h3 className="font-medium mb-2">Upload Content</h3>
              <p className="text-sm text-gray-300 mb-4">Share your maps, modules, and scenarios with the community.</p>
              <button className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                Upload
              </button>
            </div>
            
            <div className="bg-gray-700 p-4 rounded">
              <h3 className="font-medium mb-2">Explore Content</h3>
              <p className="text-sm text-gray-300 mb-4">Browse user-created content and official modules.</p>
              <button className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                Explore
              </button>
            </div>
            
            <div className="bg-gray-700 p-4 rounded">
              <h3 className="font-medium mb-2">Forums</h3>
              <p className="text-sm text-gray-300 mb-4">Discuss campaigns, rules, and game strategies with other players.</p>
              <button className="px-4 py-2 bg-indigo-600 hover:bg-indigo-700 rounded transition-colors">
                Visit Forums
              </button>
            </div>
          </div>
        </div>
      </main>
      
      {/* Footer */}
      <footer className="bg-gray-800 p-4 mt-4">
        <div className="container mx-auto text-center text-gray-400 text-sm">
          <p>DungeonMaster - Virtual Tabletop for D&D 5e</p>
          <p className="mt-1">
            <span className="mr-2">•</span>
            <a href="#" className="hover:text-indigo-400">Community</a> 
            <span className="mx-2">•</span>
            <a href="#" className="hover:text-indigo-400">Resources</a>
            <span className="mx-2">•</span>
            <a href="#" className="hover:text-indigo-400">Support</a>
            <span className="mx-2">•</span>
            <a href="#" className="hover:text-indigo-400">Privacy</a>
            <span className="mx-2">•</span>
            <a href="#" className="hover:text-indigo-400">Terms</a>
          </p>
        </div>
      </footer>
    </div>
  );
};

export default App;
'''
