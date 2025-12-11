## OpenAI ChatGPT v1.2025.330 Configure..gitattributes.For.LFS.md

**Q: How to configure .gitattributes to track LFS worthy file extensions (e.g., WAV, FBX, USD)?**

**A:** Configure `.gitattributes` so that all large binary asset types use the LFS filter, explicitly disabling text treatment.

Example `.gitattributes`:

```gitattributes
# Audio
*.wav  filter=lfs diff=lfs merge=lfs -text
*.flac filter=lfs diff=lfs merge=lfs -text
*.ogg  filter=lfs diff=lfs merge=lfs -text

# 3D assets
*.fbx  filter=lfs diff=lfs merge=lfs -text
*.obj  filter=lfs diff=lfs merge=lfs -text
*.usd  filter=lfs diff=lfs merge=lfs -text
*.usda filter=lfs diff=lfs merge=lfs -text
*.usdc filter=lfs diff=lfs merge=lfs -text

# (Unreal-adjacent, if you want them in LFS too)
*.uasset filter=lfs diff=lfs merge=lfs -text
*.umap   filter=lfs diff=lfs merge=lfs -text

# Optional: textures and high-res images
*.exr  filter=lfs diff=lfs merge=lfs -text
*.tga  filter=lfs diff=lfs merge=lfs -text
*.tif  filter=lfs diff=lfs merge=lfs -text
*.png  filter=lfs diff=lfs merge=lfs -text
*.jpg  filter=lfs diff=lfs merge=lfs -text
```

Minimal procedure

1. Install and initialize Git LFS (once per machine):

   ```bash
   git lfs install
   ```
2. Either:

   * Edit `.gitattributes` manually as above, or
   * Use `git lfs track` (which appends to `.gitattributes`):

     ```bash
     git lfs track "*.wav"
     git lfs track "*.fbx"
     git lfs track "*.usd"
     # etc.
     ```
3. Commit `.gitattributes` before committing new large files:

   ```bash
   git add .gitattributes
   git commit -m "Configure LFS for binary assets"
   ```
4. Add and commit your assets:

   ```bash
   git add path/to/assets
   git commit -m "Add assets via LFS"
   ```

If you already committed large binaries without LFS

Use `git lfs migrate` to rewrite history so those extensions are stored in LFS:

```bash
git lfs migrate import --include="*.wav,*.fbx,*.usd"
```

Run that on a branch that you can safely rewrite (typically before other collaborators are relying on the history).
