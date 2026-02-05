# OpenFOAM MCP Server for Foundry & Metallurgical Simulations

[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

An MCP (Model Context Protocol) server that enables AI agents like Claude to interact with OpenFOAM for foundry casting and metallurgical simulations.

## 🎯 Features

- **12+ Foundry-Specific Tools** for AI agents
- **Automated Case Setup** for different casting types
- **Material Database** with properties for common metals and mold materials
- **Defect Prediction** (porosity, shrinkage, hot spots, cold shuts, misruns)
- **Parametric Optimization** for gating systems
- **Multi-Physics Support** (mold filling, solidification, heat transfer)
- **Parallel Execution** support for faster simulations

## ✨ Recent Updates

### Real Physics-Based Analysis (Latest)

The result analyzer has been completely rewritten with **real physics-based calculations**:

- ✅ **Niyama Criterion** - Actual porosity prediction using temperature gradients and cooling rates
- ✅ **OpenFOAM Field Parsing** - Direct parsing of temperature (T) and volume fraction (alpha) fields
- ✅ **Parameter-Dependent Results** - Different inputs now produce different outputs (no more templated responses!)
- ✅ **Comprehensive Testing** - Full test suite with mock OpenFOAM cases verifies calculations
- ✅ **Diagnostic Tools** - Built-in health check to verify correct analyzer is active

**Key Formula:** Niyama Criterion for porosity prediction:
```
Ny = G / √R
where:
  G = Temperature gradient (K/m)
  R = Cooling rate (K/s)

Ny < 0.5  → High porosity risk
Ny > 1.0  → Low porosity risk
```

**For WSL2 users:** See [WSL2_SETUP.md](WSL2_SETUP.md) for environment configuration guide.

**Verification:** Run `python tests/test_real_analyzer.py` to see real physics calculations in action.

## 📋 Prerequisites

### Required

- **Python 3.10+**
- **OpenFOAM v11 or later** (v2306, v2312, etc.)
- **Claude Code** or other MCP-compatible client

### Optional

- **ParaView** for visualization
- **PyFoam** for advanced OpenFOAM file parsing
- **Multi-core CPU** for parallel simulations

## 🚀 Installation

### 1. Install OpenFOAM

#### Ubuntu/Debian
```bash
# Add OpenFOAM repository
sudo sh -c "wget -O - http://dl.openfoam.org/gpg.key | apt-key add -"
sudo add-apt-repository http://dl.openfoam.org/ubuntu

# Install OpenFOAM
sudo apt-get update
sudo apt-get install openfoam11

# Source OpenFOAM environment
source /opt/openfoam11/etc/bashrc

# Add to your ~/.bashrc for persistence
echo "source /opt/openfoam11/etc/bashrc" >> ~/.bashrc
```

#### Docker (Alternative)
```bash
docker pull openfoam/openfoam11-paraview512
```

### 2. Install OpenFOAM MCP Server

```bash
# Clone repository
git clone https://github.com/yourusername/openfoam-mcp.git
cd openfoam-mcp

# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\\Scripts\\activate

# Install package
pip install -e .

# Or install dependencies directly
pip install -r requirements.txt
```

### 3. Configure Claude Code

Add to your `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "openfoam": {
      "command": "python",
      "args": ["-m", "openfoam_mcp.server"],
      "env": {
        "FOAM_INST_DIR": "/opt/openfoam11"
      }
    }
  }
}
```

## 🎮 Usage

### Quick Start Example

```python
# Using Claude Code CLI
claude-code

# Then ask Claude:
> "Create a mold filling simulation for an aluminum casting at 750°C with a sand mold"

# Claude will:
# 1. Create the case
# 2. Set up geometry
# 3. Configure material properties
# 4. Generate mesh
# 5. Run simulation
# 6. Analyze results for defects
```

### Available Tools

| Tool | Description |
|------|-------------|
| `create_casting_case` | Create new OpenFOAM case |
| `list_cases` | List all cases |
| `setup_geometry` | Configure geometry (STL or parametric) |
| `setup_material_properties` | Set metal and mold properties |
| `setup_boundary_conditions` | Configure BCs |
| `run_mesh_generation` | Generate computational mesh |
| `run_simulation` | Execute OpenFOAM solver |
| `analyze_results` | Analyze simulation results |
| `predict_defects` | Predict casting defects |
| `export_results` | Export results (VTK, STL, CSV) |
| `get_case_status` | Check case status |
| `optimize_gating_system` | Optimize gate/riser positions |

### Supported Casting Types

- **Mold Filling**: Gravity pour, low-pressure, high-pressure
- **Solidification**: With heat transfer and phase change
- **Continuous Casting**: Steel, aluminum continuous casting
- **Die Casting**: High-pressure die casting

### Supported Materials

**Metals:**
- Steel (carbon steel)
- Aluminum (pure and alloys)
- Iron (cast iron)
- Copper
- Bronze

**Mold Materials:**
- Sand
- Ceramic
- Metal (permanent mold)
- Graphite

## 📊 Example Workflow

### 1. Create a Case
```python
# Claude command:
"Create a solidification simulation for steel at 1650°C in a sand mold named steel_casting_01"
```

### 2. Add Geometry
```python
# Option A: From STL file
"Set up geometry for steel_casting_01 using the STL file at /path/to/casting.stl with fine mesh"

# Option B: Parametric
"Set up a simple box geometry for steel_casting_01: 0.2m x 0.15m x 0.1m with medium mesh"
```

### 3. Run Simulation
```python
"Run the simulation for steel_casting_01 using interPhaseChangeFoam for 100 seconds"
```

### 4. Analyze Results
```python
"Analyze steel_casting_01 and predict all defects"
```

### 5. Optimize
```python
"Optimize the gating system for steel_casting_01 to minimize porosity"
```

## 🏗️ Architecture

```
┌─────────────────┐       MCP Protocol        ┌──────────────────┐
│   Claude AI     │ ←──────────────────────→  │  OpenFOAM MCP    │
│                 │                            │     Server       │
└─────────────────┘                            └──────────────────┘
                                                        ↓
                                               ┌──────────────────┐
                                               │  API Managers    │
                                               │  - OpenFOAMClient│
                                               │  - CaseManager   │
                                               │  - ResultAnalyzer│
                                               └──────────────────┘
                                                        ↓
                                               ┌──────────────────┐
                                               │  Case Builders   │
                                               │  - Templates     │
                                               │  - Material DB   │
                                               └──────────────────┘
                                                        ↓
                                               ┌──────────────────┐
                                               │  OpenFOAM Core   │
                                               │  (Solvers)       │
                                               └──────────────────┘
```

## 🧪 Testing

```bash
# Run all tests
pytest

# Run with coverage
pytest --cov=openfoam_mcp --cov-report=html

# Run specific test
pytest tests/test_case_manager.py
```

## 📁 Project Structure

```
openfoam-mcp/
├── openfoam_mcp/
│   ├── __init__.py
│   ├── server.py              # Main MCP server
│   ├── api/
│   │   ├── __init__.py
│   │   ├── openfoam_client.py  # OpenFOAM command execution
│   │   ├── case_manager.py     # Case creation/management
│   │   └── result_analyzer.py  # Result analysis
│   ├── builders/
│   │   ├── __init__.py
│   │   ├── case_builder.py     # Case file builder
│   │   └── templates.py        # OpenFOAM file templates
│   └── utils/
│       └── __init__.py
├── tests/
│   ├── test_server.py
│   ├── test_case_manager.py
│   └── test_openfoam_client.py
├── examples/
│   ├── simple_mold_filling.md
│   └── solidification_analysis.md
├── docs/
│   ├── getting_started.md
│   └── tool_reference.md
├── pyproject.toml
├── requirements.txt
└── README.md
```

## 🔧 Configuration

### Environment Variables

- `FOAM_INST_DIR`: OpenFOAM installation directory (default: `/opt/openfoam11`)
- `OPENFOAM_RUN_DIR`: Directory for simulation cases (default: `~/foam/run`)

### Custom Material Database

You can extend the material database by editing `openfoam_mcp/builders/case_builder.py`:

```python
self.metal_database["my_alloy"] = {
    "density": 8000,
    "viscosity": 0.005,
    "thermal_conductivity": 40,
    "specific_heat": 500,
    "liquidus_temp": 1700,
    "solidus_temp": 1500,
    "latent_heat": 250000
}
```

## 🤝 Contributing

Contributions welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## 📝 Roadmap

- [ ] Add more casting types (investment casting, centrifugal casting)
- [ ] Implement real-time progress monitoring
- [ ] Add machine learning-based defect prediction
- [ ] Support for more OpenFOAM solvers
- [ ] Integration with commercial casting software for validation
- [ ] Web-based visualization interface
- [ ] Automated gating system design
- [ ] Multi-objective optimization

## 🐛 Known Issues

- Mesh generation for complex geometries may require manual snappyHexMesh tuning
- Large simulations (>1M cells) require parallel execution
- Temperature-dependent material properties not yet implemented

## 📚 References

### OpenFOAM Resources
- [OpenFOAM User Guide](https://www.openfoam.com/documentation/user-guide)
- [OpenFOAM Tutorials](https://wiki.openfoam.com/Tutorials)
- [CFD Online Forums](https://www.cfd-online.com/Forums/openfoam/)

### Casting Simulation
- [directChillFoam Paper](https://joss.theoj.org/papers/10.21105/joss.04871)
- [OpenFOAM Casting Resources](https://www.researchgate.net/publication/305347635)

### MCP Protocol
- [Model Context Protocol Docs](https://modelcontextprotocol.io/)
- [MCP Python SDK](https://github.com/anthropics/mcp-python-sdk)

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🙏 Acknowledgments

- OpenFOAM Foundation for the excellent CFD toolkit
- Anthropic for the Model Context Protocol
- The foundry and metallurgy research community

## 📧 Contact

For questions or support:
- Open an issue on GitHub
- Email: your.email@example.com

---

**Built with ❤️ for the foundry and metallurgical engineering community**
