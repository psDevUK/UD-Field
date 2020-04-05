# UD-Field
Editable Input Field based on https://github.com/Garphild/react-editable-input

## Parameters
* -Id allows you to set a unique id to the input field to the obtain the values from it at a later stage
* -Label this is a mandatory parameter which displays the placeholder for the input field
* -Value allows you to display a pre-defined value in the field before it is edited
* -FullWidth can either be set to $true or $false default is $true
* -Disabled will determine if the input field can be edited

## Example
This example uses the UDSelector component I built, in-conjunction with this component to dynamically display input fields of
the selected process IDs.

```
Import-Module -Name UniversalDashboard.Community -RequiredVersion 2.8.1
Import-Module UniversalDashboard.UDSelector
Import-Module UniversalDashboard.UDField
Get-UDDashboard | Stop-UDDashboard
$theme = New-UDTheme -Name "Basic" -Definition @{
    '.css-1wa3eu0-placeholder'        = @{
        'color' = "#56587b !important"
    }
    '.css-1okebmr-indicatorSeparator' = @{
        'background-color' = "#56587b !important"
    }
    '.css-1hwfws3'                    = @{
        'height'      = "30px"
        'align-items' = "flex-start"
        'box-sizing'  = "initial !important"
        'flex-wrap'   = "initial !important"
    }
    '.css-1rhbuit-multiValue'         = @{
        'background-color' = "#323246 !important"

    }
    '.css-xb97g8'                     = @{
        'background-color' = "#56587b"
        'color'            = "#fffaf4"
    }
    '.css-12jo7m5'                    = @{
        'color' = "rgb(255, 255, 255) !important"
    }
    '.css-tlfecz-indicatorContainer'  = @{
        'color' = "#56587b !important"
    }
    '.css-yk16xz-control'             = @{
        'border-color' = "#56587b !important"
    }
    '.css-1g6gooi'                    = @{
        'padding-top' = "9px !important"
        'color'       = "#56587b !important"
    }
} -Parent "Default"
$init = New-UDEndpointInitialization -Module @('UniversalDashboard.UDSelector', 'UniversalDashboard.UDField')
Start-UDDashboard -Port 10005 -Dashboard (
    New-UDDashboard -Title "Powershell UniversalDashboard" -Theme $theme -EndpointInitialization $init -Content {
        New-UDRow -Columns {
            New-UDColumn -size 7 -Endpoint {
                $Processes = Get-Process
                $hash = @()
                foreach ($item in $Processes) {

                    $hash += @{
                        value = $item.Name
                        label = $item.Id
                    }
                }
                New-UDSelector -id  "selector1" -Options {
                    $hash
                } -PlaceHolder "Select running processes..."
            }
            New-UDColumn -Size 4 -Endpoint {
                New-UDButton -Text "Processes Selected" -OnClick {
                    function Get-UDSelectorValue {
                        param($SelectorId)
                        $value = (((Get-UDElement -Id $SelectorId).Attributes.selectedOption | Select-Object Root -ErrorAction SilentlyContinue) | ConvertTo-Json -Depth 2 | ConvertFrom-Json | Select-Object -ExpandProperty Root -ErrorAction SilentlyContinue) | Select-Object -First 1
                        if (!$value) {
                            $val = ((Get-UDElement -Id $SelectorId).Attributes.selectedOption | ConvertTo-Json -Depth 1 | ConvertFrom-Json | Select-Object -ExpandProperty SyncRoot) | Select-Object -ExpandProperty SyncRoot
                            $length = $val.length
                            $i = 0
                            Do {
                                if (($i % 2) -eq 0) {
                                    $value += "$($val[$i])" + ","
                                }
                                $i++
                            }
                            While ($i -le $length)
                            $value = $value.TrimEnd(",")
                        }
                        return $value
                    }
                    $Selected = (Get-UDSelectorValue -SelectorId 'selector1') -replace ",''"
                    if (($Selected) -notmatch "\A'" ) {
                        $Selected = "$Selected"
                    }
                    $Session:Selected = $Selected
                    Show-UDToast -Message "You selected $Session:Selected" -Duration 4000 -Position topLeft
                    $Session:Selected > C:\ud\test.txt
                    @("Results") | Sync-UDElement
                }
            }
            New-UDColumn -Size 12 -Endpoint {
                New-UDElement -Id "Results" -Tag div -Endpoint {
                    if (-not($Session:Selected -eq $null)) {
                        New-UDHeading -Size 4 -Text "You selected the following process names"
                        $procs = $Session:Selected -split ','
                        foreach ($proc in $procs) {
                            New-UDField -Label $proc -Value $proc
                        }
                    }
                } -AutoRefresh
            }
        }


    }
)

```
