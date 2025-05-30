```
import React, { useState, useEffect, useRef } from 'react';
import { Upload, Download, RotateCw, FlipHorizontal, FlipVertical, ZoomIn, ZoomOut } from 'lucide-react';
import * as Papa from 'papaparse';

const QRCodeAnalyzer = () => {
  const [data, setData] = useState([]);
  const [gridSize, setGridSize] = useState(25);
  const [threshold, setThreshold] = useState(0.5);
  const [rotation, setRotation] = useState(0);
  const [flipH, setFlipH] = useState(false);
  const [flipV, setFlipV] = useState(false);
  const [zoom, setZoom] = useState(1);
  const [qrMatrix, setQrMatrix] = useState([]);
  const canvasRef = useRef(null);
  const fileInputRef = useRef(null);

  // Sample data for demonstration
  const generateSampleData = () => {
    const sampleData = [];
    // Create a simple QR-like pattern
    const pattern = [
      [1,1,1,1,1,1,1,0,1,0,1,1,1,1,1,1,1],
      [1,0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,1],
      [1,0,1,1,1,0,1,0,1,0,1,0,1,1,1,0,1],
      [1,0,1,1,1,0,1,0,0,0,1,0,1,1,1,0,1],
      [1,0,1,1,1,0,1,0,1,0,1,0,1,1,1,0,1],
      [1,0,0,0,0,0,1,0,0,0,1,0,0,0,0,0,1],
      [1,1,1,1,1,1,1,0,1,0,1,1,1,1,1,1,1],
      [0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0],
      [1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1],
      [0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0],
      [1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1],
      [0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0],
      [1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1],
      [0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0],
      [1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1],
      [0,1,0,1,0,1,0,1,0,1,0,1,0,1,0,1,0],
      [1,1,1,1,1,1,1,0,1,0,1,1,1,1,1,1,1]
    ];
    
    for (let y = 0; y < pattern.length; y++) {
      for (let x = 0; x < pattern[y].length; x++) {
        if (pattern[y][x] === 1) {
          sampleData.push({ x, y, label: 1 });
        }
      }
    }
    return sampleData;
  };

  useEffect(() => {
    // Load sample data initially
    setData(generateSampleData());
  }, []);

  const handleFileUpload = (event) => {
    const file = event.target.files[0];
    if (file && file.type === 'text/csv') {
      Papa.parse(file, {
        complete: (results) => {
          const csvData = results.data
            .filter(row => row.label === '1' || row.label === 1)
            .map(row => ({
              x: parseFloat(row.x),
              y: parseFloat(row.y),
              label: parseInt(row.label)
            }))
            .filter(row => !isNaN(row.x) && !isNaN(row.y));
          setData(csvData);
        },
        header: true,
        skipEmptyLines: true
      });
    }
  };

  const processDataToGrid = () => {
    if (data.length === 0) return [];

    // Find bounds
    const xValues = data.map(d => d.x);
    const yValues = data.map(d => d.y);
    const minX = Math.min(...xValues);
    const maxX = Math.max(...xValues);
    const minY = Math.min(...yValues);
    const maxY = Math.max(...yValues);

    // Create grid
    const grid = Array(gridSize).fill().map(() => Array(gridSize).fill(0));
    
    data.forEach(point => {
      const gridX = Math.floor(((point.x - minX) / (maxX - minX)) * (gridSize - 1));
      const gridY = Math.floor(((point.y - minY) / (maxY - minY)) * (gridSize - 1));
      
      if (gridX >= 0 && gridX < gridSize && gridY >= 0 && gridY < gridSize) {
        grid[gridY][gridX] = 1;
      }
    });

    return grid;
  };

  const applyTransformations = (grid) => {
    let transformed = [...grid.map(row => [...row])];

    // Apply rotation
    for (let i = 0; i < rotation; i++) {
      const newGrid = Array(transformed[0].length).fill().map(() => Array(transformed.length).fill(0));
      for (let y = 0; y < transformed.length; y++) {
        for (let x = 0; x < transformed[0].length; x++) {
          newGrid[x][transformed.length - 1 - y] = transformed[y][x];
        }
      }
      transformed = newGrid;
    }

    // Apply horizontal flip
    if (flipH) {
      transformed = transformed.map(row => row.reverse());
    }

    // Apply vertical flip
    if (flipV) {
      transformed = transformed.reverse();
    }

    return transformed;
  };

  useEffect(() => {
    const grid = processDataToGrid();
    const transformed = applyTransformations(grid);
    setQrMatrix(transformed);
  }, [data, gridSize, rotation, flipH, flipV]);

  const drawCanvas = () => {
    const canvas = canvasRef.current;
    if (!canvas || qrMatrix.length === 0) return;

    const ctx = canvas.getContext('2d');
    const cellSize = (400 / qrMatrix.length) * zoom;
    
    canvas.width = qrMatrix[0].length * cellSize;
    canvas.height = qrMatrix.length * cellSize;
    
    ctx.fillStyle = 'white';
    ctx.fillRect(0, 0, canvas.width, canvas.height);
    
    ctx.fillStyle = 'black';
    for (let y = 0; y < qrMatrix.length; y++) {
      for (let x = 0; x < qrMatrix[y].length; x++) {
        if (qrMatrix[y][x] === 1) {
          ctx.fillRect(x * cellSize, y * cellSize, cellSize, cellSize);
        }
      }
    }
  };

  useEffect(() => {
    drawCanvas();
  }, [qrMatrix, zoom]);

  const downloadQRCode = () => {
    const canvas = canvasRef.current;
    const link = document.createElement('a');
    link.download = 'qr_code.png';
    link.href = canvas.toDataURL();
    link.click();
  };

  const generateQRString = () => {
    return qrMatrix.map(row => row.join('')).join('\n');
  };

  return (
    <div className="max-w-6xl mx-auto p-6 bg-gray-50 min-h-screen">
      <div className="bg-white rounded-lg shadow-lg p-6">
        <h1 className="text-3xl font-bold text-gray-800 mb-6">QR Code Pattern Analyzer</h1>
        
        {/* File Upload */}
        <div className="mb-6">
          <input
            ref={fileInputRef}
            type="file"
            accept=".csv"
            onChange={handleFileUpload}
            className="hidden"
          />
          <button
            onClick={() => fileInputRef.current?.click()}
            className="flex items-center gap-2 bg-blue-500 hover:bg-blue-600 text-white px-4 py-2 rounded-lg transition-colors"
          >
            <Upload size={20} />
            Upload CSV File
          </button>
          <p className="text-sm text-gray-600 mt-2">
            Upload a CSV file with columns: x, y, label (where label=1 for QR dots)
          </p>
        </div>

        <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
          {/* Controls */}
          <div className="space-y-4">
            <h2 className="text-xl font-semibold text-gray-700">Controls</h2>
            
            {/* Grid Size */}
            <div>
              <label className="block text-sm font-medium text-gray-700 mb-2">
                Grid Size: {gridSize}x{gridSize}
              </label>
              <input
                type="range"
                min="10"
                max="50"
                value={gridSize}
                onChange={(e) => setGridSize(parseInt(e.target.value))}
                className="w-full"
              />
            </div>

            {/* Transformations */}
            <div className="flex flex-wrap gap-2">
              <button
                onClick={() => setRotation((rotation + 1) % 4)}
                className="flex items-center gap-2 bg-green-500 hover:bg-green-600 text-white px-3 py-2 rounded-lg"
              >
                <RotateCw size={16} />
                Rotate 90°
              </button>
              <button
                onClick={() => setFlipH(!flipH)}
                className={`flex items-center gap-2 px-3 py-2 rounded-lg text-white ${
                  flipH ? 'bg-orange-600' : 'bg-orange-500 hover:bg-orange-600'
                }`}
              >
                <FlipHorizontal size={16} />
                Flip H
              </button>
              <button
                onClick={() => setFlipV(!flipV)}
                className={`flex items-center gap-2 px-3 py-2 rounded-lg text-white ${
                  flipV ? 'bg-purple-600' : 'bg-purple-500 hover:bg-purple-600'
                }`}
              >
                <FlipVertical size={16} />
                Flip V
              </button>
            </div>

            {/* Zoom */}
            <div className="flex items-center gap-2">
              <button
                onClick={() => setZoom(Math.max(0.5, zoom - 0.25))}
                className="flex items-center gap-2 bg-gray-500 hover:bg-gray-600 text-white px-3 py-2 rounded-lg"
              >
                <ZoomOut size={16} />
              </button>
              <span className="text-sm text-gray-700">Zoom: {zoom}x</span>
              <button
                onClick={() => setZoom(Math.min(3, zoom + 0.25))}
                className="flex items-center gap-2 bg-gray-500 hover:bg-gray-600 text-white px-3 py-2 rounded-lg"
              >
                <ZoomIn size={16} />
              </button>
            </div>

            {/* Download */}
            <button
              onClick={downloadQRCode}
              className="flex items-center gap-2 bg-indigo-500 hover:bg-indigo-600 text-white px-4 py-2 rounded-lg"
            >
              <Download size={20} />
              Download QR Code
            </button>

            {/* Stats */}
            <div className="bg-gray-100 p-4 rounded-lg">
              <h3 className="font-semibold text-gray-700 mb-2">Statistics</h3>
              <p className="text-sm text-gray-600">Data Points: {data.length}</p>
              <p className="text-sm text-gray-600">Grid Size: {gridSize}×{gridSize}</p>
              <p className="text-sm text-gray-600">Black Modules: {qrMatrix.flat().filter(x => x === 1).length}</p>
            </div>
          </div>

          {/* QR Code Display */}
          <div className="space-y-4">
            <h2 className="text-xl font-semibold text-gray-700">QR Code Pattern</h2>
            <div className="border-2 border-gray-300 rounded-lg p-4 bg-white overflow-auto max-h-96">
              <canvas
                ref={canvasRef}
                className="border border-gray-200"
                style={{ imageRendering: 'pixelated' }}
              />
            </div>
            
            {/* Matrix Output */}
            <div className="bg-gray-100 p-4 rounded-lg">
              <h3 className="font-semibold text-gray-700 mb-2">Matrix (for debugging)</h3>
              <textarea
                value={generateQRString()}
                readOnly
                className="w-full h-32 text-xs font-mono bg-white border border-gray-300 rounded p-2"
              />
            </div>
          </div>
        </div>

        {/* Tips */}
        <div className="mt-6 bg-blue-50 border-l-4 border-blue-400 p-4">
          <h3 className="font-semibold text-blue-800 mb-2">Tips for QR Code Recovery:</h3>
          <ul className="text-blue-700 text-sm space-y-1">
            <li>• Adjust grid size to match the expected QR code dimensions (21×21, 25×25, etc.)</li>
            <li>• Try different rotations - QR codes can be read from any orientation</li>
            <li>• Use flips if the pattern appears mirrored</li>
            <li>• QR codes have distinctive square patterns in three corners</li>
            <li>• If the pattern looks correct but won't scan, try different QR readers</li>
          </ul>
        </div>
      </div>
    </div>
  );
};

export default QRCodeAnalyzer;
```