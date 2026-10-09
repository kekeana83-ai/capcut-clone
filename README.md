# capcut-clone
CapCut-inspired video editor web application
import { useMemo, useRef, useState } from 'react';

const filters = {
  none: 'none',
  vivid: 'contrast(1.15) saturate(1.35) brightness(1.05)',
  cinematic: 'contrast(1.25) saturate(0.8) brightness(0.95)',
  neon: 'contrast(1.2) saturate(1.6) hue-rotate(18deg)',
  mono: 'grayscale(1) contrast(1.1) brightness(1.05)'
};

const defaultClips = [
  {
    id: 1,
    name: 'Intro',
    duration: 8,
    trimStart: 0,
    trimEnd: 8,
    filter: 'vivid'
  },
  {
    id: 2,
    name: 'Montagem',
    duration: 12,
    trimStart: 0,
    trimEnd: 12,
    filter: 'cinematic'
  },
  {
    id: 3,
    name: 'CTA',
    duration: 6,
    trimStart: 0,
    trimEnd: 6,
    filter: 'neon'
  }
];

function App() {
  const fileInputRef = useRef(null);
  const [projectName, setProjectName] = useState('Vibe Cut Pro');
  const [clips, setClips] = useState(defaultClips);
  const [selectedClipId, setSelectedClipId] = useState(defaultClips[0].id);
  const [filterMode, setFilterMode] = useState('vivid');
  const [isPlaying, setIsPlaying] = useState(false);
  const [textOverlay, setTextOverlay] = useState({
    text: 'SEU VIDEO',
    x: 50,
    y: 18,
    size: 58,
    color: '#f8d76b'
  });

  const selectedClip = useMemo(
    () => clips.find((clip) => clip.id === selectedClipId) ?? clips[0],
    [clips, selectedClipId]
  );

  const totalDuration = useMemo(
    () => clips.reduce((sum, clip) => sum + Math.max(clip.trimEnd - clip.trimStart, 0), 0),
    [clips]
  );

  const updateSelectedClip = (updates) => {
    if (!selectedClip) return;

    setClips((currentClips) =>
      currentClips.map((clip) =>
        clip.id === selectedClip.id ? { ...clip, ...updates } : clip
      )
    );
  };

  const handleUpload = (event) => {
    const files = Array.from(event.target.files || []);

    if (!files.length) return;

    const newClips = files.map((file, index) => {
      const objectUrl = URL.createObjectURL(file);
      return {
        id: Date.now() + index + Math.random(),
        name: file.name.replace(/\.[^/.]+$/, ''),
        src: objectUrl,
        duration: 10,
        trimStart: 0,
        trimEnd: 10,
        filter: 'none'
      };
    });

    setClips((current) => [...current, ...newClips]);
    setSelectedClipId(newClips[0].id);
    event.target.value = '';
  };

  const handleTrim = (field, value) => {
    const nextValue = Number(value);

    if (field === 'trimStart') {
      updateSelectedClip({
        trimStart: Math.min(nextValue, selectedClip.trimEnd - 1 || nextValue),
        duration: Math.max(selectedClip.trimEnd - nextValue, 1)
      });
    }

    if (field === 'trimEnd') {
      updateSelectedClip({
        trimEnd: Math.max(nextValue, selectedClip.trimStart + 1),
        duration: Math.max(nextValue - selectedClip.trimStart, 1)
      });
    }
  };

  const addTextPreset = () => {
    const presets = ['SEU VIDEO', 'VIRE O MODELO', 'SALVE AGORA', 'OUTRO NIVEL'];
    const nextIndex = clips.length % presets.length;
    setTextOverlay((current) => ({
      ...current,
      text: presets[nextIndex]
    }));
  };

  return (
    <div className="app-shell">
      <header className="topbar">
        <div className="brand-wrap">
          <div className="brand-mark">C</div>
          <div>
            <p className="eyebrow">Editor</p>
            <h1>{projectName}</h1>
          </div>
        </div>

        <div className="toolbar-actions">
          <input
            value={projectName}
            onChange={(event) => setProjectName(event.target.value)}
            className="project-name-input"
            aria-label="Nome do projeto"
          />
          <button className="secondary-button">Salvar</button>
          <button className="primary-button">Exportar</button>
        </div>
      </header>

      <div className="workspace">
        <aside className="panel sidebar left-panel">
          <div className="panel-block">
            <p className="section-label">Ferramentas</p>
            <button className="tool-button active">Cortar</button>
            <button className="tool-button">Texto</button>
            <button className="tool-button">Filtros</button>
            <button className="tool-button">Transição</button>
            <button className="tool-button">Música</button>
          </div>

          <div className="panel-block">
            <p className="section-label">Importar</p>
            <button className="tool-button primary-fill" onClick={() => fileInputRef.current?.click()}>
              + Adicionar vídeo
            </button>
            <input
              ref={fileInputRef}
              type="file"
              accept="video/*"
              multiple
              hidden
              onChange={handleUpload}
            />
          </div>
        </aside>

        <main className="studio-panel">
          <div className="preview-header">
            <div>
              <p className="section-label">Preview</p>
              <h2>{selectedClip?.name ?? 'Novo vídeo'}</h2>
            </div>
            <div className="preview-controls">
              <button className="secondary-button" onClick={() => setIsPlaying((value) => !value)}>
                {isPlaying ? 'Pausar' : 'Play'}
              </button>
            </div>
          </div>

          <div className="stage-wrap">
            <div className="preview-stage">
              {selectedClip?.src ? (
                <video
                  className="video-preview"
                  src={selectedClip.src}
                  muted={!isPlaying}
                  autoPlay={isPlaying}
                  loop={isPlaying}
                  playsInline
                  style={{ filter: filters[filterMode] || 'none' }}
                />
              ) : (
                <div className="video-placeholder">
                  <div className="placeholder-icon">▶</div>
                  <p>Adicione vídeos para começar a edição</p>
                </div>
              )}

              <div
                className="text-overlay"
                style={{
                  left: `${textOverlay.x}%`,
                  top: `${textOverlay.y}%`,
                  fontSize: `${textOverlay.size}px`,
                  color: textOverlay.color
                }}
              >
                {textOverlay.text}
              </div>
            </div>
          </div>

          <div className="timeline-panel">
            <div className="timeline-topbar">
              <p className="section-label">Timeline</p>
              <span>{totalDuration.toFixed(1)}s</span>
            </div>

            <div className="timeline-items">
              {clips.map((clip) => (
                <button
                  key={clip.id}
                  className={`timeline-item ${selectedClipId === clip.id ? 'active' : ''}`}
                  onClick={() => setSelectedClipId(clip.id)}
                >
                  <span className="timeline-item-name">{clip.name}</span>
                  <span>{Math.max(clip.trimEnd - clip.trimStart, 0).toFixed(1)}s</span>
                </button>
              ))}
            </div>

            {selectedClip && (
              <div className="trim-panel">
                <label>
                  Início
                  <input
                    type="range"
                    min="0"
                    max={Math.max(selectedClip.trimEnd - 1, 0)}
                    value={selectedClip.trimStart}
                    onChange={(event) => handleTrim('trimStart', event.target.value)}
                  />
                </label>
                <label>
                  Fim
                  <input
                    type="range"
                    min={selectedClip.trimStart + 1}
                    max={Math.max(selectedClip.duration, selectedClip.trimStart + 1)}
                    value={selectedClip.trimEnd}
                    onChange={(event) => handleTrim('trimEnd', event.target.value)}
                  />
                </label>
              </div>
            )}
          </div>
        </main>

        <aside className="panel sidebar right-panel">
          <div className="panel-block">
            <p className="section-label">Efeitos</p>
            <label>
              Filtro
              <select value={filterMode} onChange={(event) => setFilterMode(event.target.value)}>
                <option value="none">Padrão</option>
                <option value="vivid">Vivid</option>
                <option value="cinematic">Cinematic</option>
                <option value="neon">Neon</option>
                <option value="mono">Mono</option>
              </select>
            </label>
          </div>

          <div className="panel-block">
            <p className="section-label">Texto</p>
            <input
              value={textOverlay.text}
              onChange={(event) => setTextOverlay((current) => ({ ...current, text: event.target.value }))}
              placeholder="Digite o texto"
            />
            <label>
              Tamanho
              <input
                type="range"
                min="24"
                max="120"
                value={textOverlay.size}
                onChange={(event) =>
                  setTextOverlay((current) => ({ ...current, size: Number(event.target.value) }))
                }
              />
            </label>
            <label>
              Cor
              <input
                type="color"
                value={textOverlay.color}
                onChange={(event) =>
                  setTextOverlay((current) => ({ ...current, color: event.target.value }))
                }
              />
            </label>
            <input
              type="range"
              min="10"
              max="90"
              value={textOverlay.x}
              onChange={(event) =>
                setTextOverlay((current) => ({ ...current, x: Number(event.target.value) }))
              }
              aria-label="Posição horizontal do texto"
            />
            <input
              type="range"
              min="10"
              max="85"
              value={textOverlay.y}
              onChange={(event) =>
                setTextOverlay((current) => ({ ...current, y: Number(event.target.value) }))
              }
              aria-label="Posição vertical do texto"
            />
            <button className="secondary-button full-width" onClick={addTextPreset}>
              Usar preset
            </button>
          </div>
        </aside>
      </div>
    </div>
  );
}

export default App;
