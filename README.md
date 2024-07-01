# MarchingCubesTables.h

This is taken from user https://gist.github.com/dwilliamson who had a JavaScript file for the marching cubes tables. I needed it for C++, so there you go.

Converted from JS to C++ from the following gist: https://gist.githubusercontent.com/dwilliamson/c041e3454a713e58baf6e4f8e5fffecd/raw/3c761fa2dfc4333749626ed3d1a0ff25d708089c/MarchingCubes.js

## Example usage

```c++
#include "MarchingCubesTables.h"

struct Triangle {
    glm::vec3 vertices[3];
    glm::vec3 normal;
};

std::vector<Triangle> GenerateMarchingCubesTriangles(const VoxelChunkComponent& chunk, float isoLevel) {
    std::vector<Triangle> triangles;

    for (int x = 0; x < chunk.width - 1; x++) {
        for (int y = 0; y < chunk.height - 1; y++) {
            for (int z = 0; z < chunk.depth - 1; z++) {
                float cube[8];
                glm::vec3 normals[8];
                glm::vec3 positions[8];

                // Sample the 8 corners of the cube
                for (int i = 0; i < 8; i++) {
                    int dx = i & 1;
                    int dy = (i & 2) >> 1;
                    int dz = (i & 4) >> 2;
                    cube[i] = VoxelHelpers::GetVoxel(chunk, x + dx, y + dy, z + dz);
                    normals[i] = VoxelHelpers::GetNormal(chunk, x + dx, y + dy, z + dz);
                    positions[i] = glm::vec3(x + dx, y + dy, z + dz);
                }

                // Determine the index in the edge table
                int cubeIndex = 0;
                if (cube[0] < isoLevel) cubeIndex |= 1;
                if (cube[1] < isoLevel) cubeIndex |= 2;
                if (cube[2] < isoLevel) cubeIndex |= 4;
                if (cube[3] < isoLevel) cubeIndex |= 8;
                if (cube[4] < isoLevel) cubeIndex |= 16;
                if (cube[5] < isoLevel) cubeIndex |= 32;
                if (cube[6] < isoLevel) cubeIndex |= 64;
                if (cube[7] < isoLevel) cubeIndex |= 128;

                // Skip this cube if it's entirely inside or outside the isosurface
                if (EdgeMasks[cubeIndex] == 0) continue;

                glm::vec3 vertlist[12];
                glm::vec3 normlist[12];

                // Find the vertices where the surface intersects the cube
                for (int i = 0; i < 12; i++) {
                    if (EdgeMasks[cubeIndex] & (1 << i)) {
                        int v1 = EdgeVertexIndices[i][0];
                        int v2 = EdgeVertexIndices[i][1];
                        float t = (isoLevel - cube[v1]) / (cube[v2] - cube[v1]);
                        vertlist[i] = positions[v1] + t * (positions[v2] - positions[v1]);
                        normlist[i] = glm::normalize(normals[v1] + t * (normals[v2] - normals[v1]));
                    }
                }

                // Create the triangles
                for (int i = 0; TriangleTable[cubeIndex][i] != -1; i += 3) {
                    Triangle tri;
                    tri.vertices[0] = vertlist[TriangleTable[cubeIndex][i]];
                    tri.vertices[1] = vertlist[TriangleTable[cubeIndex][i+1]];
                    tri.vertices[2] = vertlist[TriangleTable[cubeIndex][i+2]];
                    tri.normal = glm::normalize(normlist[TriangleTable[cubeIndex][i]] +
                                                normlist[TriangleTable[cubeIndex][i+1]] +
                                                normlist[TriangleTable[cubeIndex][i+2]]);
                    triangles.push_back(tri);
                }
            }
        }
    }

    return triangles;
}
```
