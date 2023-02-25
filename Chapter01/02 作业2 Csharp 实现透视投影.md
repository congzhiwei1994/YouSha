此作业基于unity实现。

主要用到的unity功能， Matrix4x4 API

代码路径：
Assets/Scripts/CustormTransformMatrix/TransformTool.cs

**model矩阵**

```c#
/// <summary>  
/// model矩阵  
/// </summary>  
public static Matrix4x4 GetModelMatrix(Vector3 scale, Vector3 rotation, Vector3 position)  
{  
    var scaleMatrix = GetScaleMatrix(scale);  
  
    var rotX = GetRotationMatrix(Vector3.right, -rotation.x);  
    var rotY = GetRotationMatrix(Vector3.up, -rotation.y);  
    var rotZ = GetRotationMatrix(Vector3.forward, rotation.z);  
    var rotMatrix = rotY * rotX * rotZ;  
  
    var translationMatrix = GetTranslationMatrix(position);  
  
    return translationMatrix * rotMatrix * scaleMatrix;  
}
```


**View矩阵**

```c#
/// <summary>  
/// view矩阵，右手坐标系，相机看向负Z  
/// </summary>  
public static Matrix4x4 GetViewMatrix(Vector3 eye_pos, Vector3 lookAtDir, Vector3 upDir)  
{  
    Matrix4x4 matrix = Matrix4x4.identity;  
  
    eye_pos.z *= -1;  
    Vector3 camZ = lookAtDir.normalized;  
    camZ.z *= -1;  
    Vector3 camY = upDir.normalized;  
    camY.z *= -1;  
    Vector3 camX = Vector3.Cross(camY, camZ);  
    camY = Vector3.Cross(camZ, camX);  
  
    matrix.SetRow(0, new Vector4(camX.x, camX.y, camX.z, 0));  
    matrix.SetRow(1, new Vector4(camY.x, camY.y, camY.z, 0));  
    matrix.SetRow(2, new Vector4(camZ.x, camZ.y, camZ.z, 0));  
    matrix.SetRow(3, new Vector4(0, 0, 0, 1));  
  
    Matrix4x4 translate = Matrix4x4.identity;  
    translate.SetColumn(3, new Vector4(-eye_pos.x, -eye_pos.y, -eye_pos.z, 1));  
  
    Matrix4x4 view = matrix * translate;  
    return view;  
}
```

**投影矩阵**

```c#
/// <summary>  
/// 投影矩阵  
/// </summary>  
public static Matrix4x4 GetProjectionMatrix(Camera camera, float aspect)  
{  
    Matrix4x4 projectionMatrix = Matrix4x4.identity;  
    if (camera.orthographic)  
    {  
        float height = camera.orthographicSize;  
        float width = height * aspect;  
        float f = -camera.farClipPlane;  
        float n = -camera.nearClipPlane;  
  
        projectionMatrix = GetOrthographicProjectionMatrix(-width, width, -height, height, f, n);  
    }  
    else  
    {  
        projectionMatrix = GetPerspectiveProjectionMatrix(camera.fieldOfView, aspect, camera.nearClipPlane,  
            camera.farClipPlane);  
    }  
  
    return projectionMatrix;  
}
```

此处投影矩阵包含正交投影和透视投影

**正交投影**
```c#
/// <summary>  
/// 正交投影 /// </summary>  
public static Matrix4x4 GetOrthographicProjectionMatrix(float l, float r, float b, float t, float f, float n)  
{  
    Matrix4x4 translate = Matrix4x4.identity;  
    translate.SetColumn(3, new Vector4(-(r + l) * 0.5f, -(t + b) * 0.5f, -(n + f) * 0.5f, 1f));  
  
    var scale = GetScaleMatrix(new Vector3(2 / (r - l), 2 / (t - b), 2 / (n - f)));  
  
    return scale * translate;  
}
```

**透视投影**

```c#
/// <summary>  
/// 透视投影  
/// </summary>  
public static Matrix4x4 GetPerspectiveProjectionMatrix(float fov, float aspect, float zNear, float zFar)  
{  
    float t = Mathf.Tan(fov * 0.5f * D2R);  
    float b = -t;  
    float r = aspect * t;  
    float l = -r;  
    float n = -zNear;  
    float f = -zFar;  
  
    Matrix4x4 perspToOrtho = Matrix4x4.identity;  
    perspToOrtho.m00 = n;  
    perspToOrtho.m11 = n;  
    perspToOrtho.m22 = n + f;  
    perspToOrtho.m23 = -n * f;  
    perspToOrtho.m32 = 1;  
    perspToOrtho.m33 = 0;  
  
    var orthoProj = GetOrthographicProjectionMatrix(l, r, b, t, f, n);  
    return orthoProj * perspToOrtho;  
}
```