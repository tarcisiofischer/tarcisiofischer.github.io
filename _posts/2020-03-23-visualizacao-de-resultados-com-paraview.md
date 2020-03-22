---
layout: post
title: Exportando resultados de malhas bidimensionais com Python para o Paraview
---

O Paraview é um software open-source multiplataforma para visualização de dados. Diferente da matplotlib,
ele é um software standalone, independente do Python (Não é uma biblioteca para visualização). A vantagem
é que é possível interagir com ele, fazer recortes de malha, filtros, entre outros. É um software
bastante útil de visualização geral de malhas (Embora existam alternativas pagas mais poderosas, para nichos
mais específicos). Eu utilizo ele as vezes para fazer análises de resultados em outputs de simuladores
relacionados à análise estrutural e CFD.

{:refdef: style="text-align: center;"}
![](/images/post7-example001.png)
{: refdef}

Acredito ser uma boa alternativa para se utilizar no dia-a-dia até pela
praticidade. O software utiliza o VTK (Visualization Toolkit), uma biblioteca opensource para visualização
2D e 3D. Uma das interfaces disponíveis é para Python. Nesse post vou mostrar uma forma de salvar arquivos
de malha não estruturada bidimensional gerado a partir de um código em Python e abrir o mesmo no Paraview.
Eu uso isso para meus projetos pessoais quando as malhas começam a ficar muito grandes e a matplotlib
começa a "apanhar".


# Malhas bidimensionais

Como comentado, para esse artigo, serão utilizadas malhas não estruturadas bidimensionais, com propriedades
nos nós dos elementos/volumes de controle. Para simplificar, utilizarei a função meshgrid da Numpy, para criar uma
malha ilustrativa com um campo de propriedades escalares. Na verdade, a função meshgrid retorna
apenas a posição dos pontos no espaço. Por isso, preciso adicionalmente de um loop para conectar os
mesmos. O código final é mostrado abaixo. Apesar de, no fim, essa função gerar uma malha estruturada,
teóricamente se estivessemos uma malha não estruturada, o arquivo do VTK que iremos
salvar continuaria funcionando.

{% highlight python linenos %}
def build_2d_quad_mesh(size_x, size_y, nx, ny):
    x = np.linspace(0, size_x, nx + 1)
    y = np.linspace(0, size_y, ny + 1)
    xv, yv = np.meshgrid(x, y)
    points = np.vstack([xv.ravel(), yv.ravel()]).T

    from_ij_to_flat_index = lambda i, j: (nx + 1) * i + j
    elements = []
    for i in range(ny):
        for j in range(nx):
            elements.append(
                [
                    from_ij_to_flat_index(i, j),
                    from_ij_to_flat_index(i, j + 1),
                    from_ij_to_flat_index(i + 1, j + 1),
                    from_ij_to_flat_index(i + 1, j),
                ]
            )
    elements = np.array(elements)

    return Mesh(nx, ny, points, elements)
{% endhighlight %}

No VTK, existe a estrutura de dados vtkUnstructuredGrid, cujo objeto pode ser exportado para um arquivo
de extensão VTU fácilmente pelo vtkXMLUnstructuredGridWriter. Essa estrutura é composta por um conjunto
de pontos e um conjunto de poliedros. Cada poliedro é composto por um conjunto de faces que, por sua vez,
é composto por um conjunto de identificadores para um ponto na lista de pontos. Como no nosso
caso queremos apenas um objeto bidimensional, forçamos todos os pontos a estarem no mesmo plano no eixo
Z, e o conversor da nossa malha de pontos anterior para a do VTK fica da seguinte forma:

{% highlight python linenos %}
import vtk

if points.shape[1] == 2:
    points = np.hstack((points, np.zeros((points.shape[0],1))))

unstructured_grid = UnstructuredGrid(vtk.vtkUnstructuredGrid())
unstructured_grid.SetPoints(points)
for polyhedron in polyhedrons:
    face_stream = vtk.vtkIdList()
    face_stream.InsertNextId(len(polyhedron))
    for face in polyhedron:
        face_stream.InsertNextId(len(face))
        for point_id in face:
            face_stream.InsertNextId(point_id)
    unstructured_grid.InsertNextCell(vtk.VTK_POLYHEDRON, face_stream)
{% endhighlight %}

{:refdef: style="text-align: center;"}
![](/images/post7-example002.png)
{: refdef}

# Propriedades escalares nodais

A fim de exemplo, criei uma função que gera um conjunto de propriedades randomico, utilizando a função
random do Python, mostrada abaixo. Os resultados posteriores, porém, são na verdade resultados de um
projeto pessoal, que é resultado de um programa de elementos finitos. Resolvi deixar a função randomica
disponível aqui apenas para aqueles que querem juntar os pedaços e testar em casa ;)

{% highlight python linenos %}
{% endhighlight %}

Finalmente, para mapear essas propriedades para o VTK, pegamos o conjunto de pontos da malha não
estruturada e atribuimos os escalares individualmente para cada nó. Aqui, dei o nome desse conjunto de
propriedades "Pressure", por que, na verdade, extraí esse código de uma função real de um projeto pessoal.
No Paraview, esse nome aparece como nome da propriedade em uma lista para o usuário escolher. Isso é
bastante útil para a identificação de cada uma das propriedades sendo visualizada.

{% highlight python linenos %}
property_name = "Pressure"
property_data_array = vtk.vtkFloatArray()
property_data_array.SetName(property_name)
for value in pressure.flatten():
    property_data_array.InsertNextValue(value)
point_data = unstructured_grid.GetPointData()
point_data.SetActiveScalars(property_name)
point_data.SetScalars(property_data_array)
{% endhighlight %}


# Salvando as propridades da malha não estruturada

Por fim, a parte mais simples é a de salvar o nosso trabalho em um arquivo VTU, que pode ser carregado no
Paraview. Como o código é muito simples, simplesmente deixo disponível abaixo para referência. Note que
esse não é o único formato possível. A biblioteca possui writers para arquivos VTK, VTU, OBJ, PLY, JSON,
entre outros que eu nem conheço (Nunca precisei ir tão a fundo assim nessa parte de vizualização).

{% highlight python linenos %}
writer = vtk.vtkXMLUnstructuredGridWriter()
writer.SetDataModeToAscii()
writer.SetCompressorTypeToNone()
writer.SetFileName(filename)
writer.SetInputData(unstructured_grid.VTKObject)
writer.Write()
{% endhighlight %}

O resultado final, no Paraview, é mostrado abaixo. Como as propriedades são posicionadas nos nós, o
software faz uma interpolação para mostrar essa distribuição dentro de cada elemento (Por isso acontece
esses degrades entro dos elementos). Apesar de eu não ter mostrado aqui, é também possível escrever dados
diretamente para cada elemento.

{:refdef: style="text-align: center;"}
![](/images/post7-example003.png)
{: refdef}

O Paraview é uma ferramenta suficientemente poderosa para visualização de grande quantidade de dados em 
malhas bi e tridimensionais gratuita e open-source. Nesse artigo, mostrei brevemente uma forma de salvar
dados nos nós de uma malha não estruturada bidimensional, para visualização no mesmo. Além da visualização,
o VTK pode ser utilizado diretamente para operações programadas, diretamente nas malhas (Ou até mesmo no
processo de geração de malhas). É uma boa alternativa gratuita para projetos pequenos ou acadêmicos.

Até a próxima!
